---
title: "8. Gain calibration"
parent: Reduction steps
nav_order: 8
permalink: /steps/08-gain/
---

# Step 8. Gain calibration (G1)
{: .no_toc }

Pipeline stage: `gaincal` · CASA task: `gaincal` with `calmode='ap'`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

Now we measure how each antenna's amplitude and phase **change with time**. We solve one amplitude and one phase per antenna, per polarisation, **per scan**, on three sources:

1. **J1939-6342** (the flux calibrator, 3 scans),
2. **PKS1313-333** (the gain calibrator, observed every ~15 minutes all night),
3. **J1311-2216** (the spare gain calibrator, also every ~15 minutes).

All three sets of solutions go into **one table**, `G1`.

## Why we need it

The atmosphere (mainly the **ionosphere**, a layer of charged particles high above the Earth) and the electronics change slowly during the night. These changes are mostly the same at all frequencies, so we do not need to solve them channel by channel; one value per scan is enough.

We cannot measure these changes on M83 itself, because we do not know what M83 looks like (that is what we want to find out!). So we measure them on **PKS1313-333**, which is a simple, bright point source **close to M83 on the sky** (about 4 degrees away). Its light passes through almost the same atmosphere as M83's light. Then we **interpolate** (estimate between measurements) the gains for the M83 scans in between.

## A trick: solving against a 1 Jy model

We set a proper flux model only for J1939-6342 (step 4). For PKS1313-333 and J1311-2216, CASA assumes a **1 Jy point source** at the centre. So:

- the **phases** of their gain solutions are correct (a point source at the centre has zero phase, whatever its brightness);
- the **amplitudes** are wrong by a factor √(S / 1 Jy), where S is the true flux of the source.

We fix the amplitudes in [step 9](../09-fluxscale/), by comparing them with the J1939-6342 solutions. This is the standard way to calibrate: one flux calibrator sets the scale, and a nearby gain calibrator follows the changes.

## What we ran

```python
common = dict(vis=msfile, caltable=gtable1, solint='inf', refant='m059', spw='0:1250~1800',
              gaintype='G', calmode='ap', minsnr=5, gaintable=[ktable0, btable0],
              interp=['nearest', 'nearest'])
gaincal(field='J1939-6342', solnorm=False, **common)
gaincal(field='PKS1313-333', append=True, **common)
gaincal(field='J1311-2216', append=True, **common)
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `calmode` | `'ap'` | Solve **a**mplitude and **p**hase. |
| `gaintype` | `'G'` | Separate solutions for X and Y. |
| `solint` | `'inf'` | One solution per scan. Each gain calibrator scan is 1–2 minutes long. The gains hardly change within 2 minutes, and a whole scan gives a high SNR. |
| `spw` | `'0:1250~1800'` | Use the good channels only. All 551 channels are averaged together (after the bandpass is applied), which gives a very precise solution. |
| `gaintable` | `[ktable0, btable0]` | Apply delays and bandpass first. **G0 is not used here**; G1 replaces it. |
| `append` | `True` | Add the solutions to the same table instead of overwriting it. Step 9 needs all three sources in one table. |
| `solnorm` | `False` | Keep the true amplitudes. We need them for the flux scale. |

## The plots

Each source has its own folder of plots: amplitude and phase against time, for all antennas together and for each antenna separately.

### J1939-6342 (flux calibrator)

{% include fig.html src="07_gain_cal_G1_fluxcal_J1939-6342/G1_J1939-6342_amp_vs_time_all_antennas.png" caption="Gain amplitudes on J1939-6342, all antennas. Colours are antennas." %}

{% include fig.html src="07_gain_cal_G1_fluxcal_J1939-6342/G1_J1939-6342_amp_vs_time_per_antenna_m000-m008.png" caption="The same, one panel per antenna (m000 to m008). Black and pink are X and Y." %}

{% include gallery.html match="G1_J1939-6342_amp_vs_time_per_antenna" title="Show amplitude plots for all antennas" %}

**What we see:** the amplitudes are between about 0.55 and 1.0. Each antenna stays almost exactly the same over the three scans (changes of 1–2%). The spread between antennas (0.55 to 1.0) is simply because each antenna has its own amplification. Calibration divides this out.

{: .ours }
> One antenna has a gain amplitude near 1.0, clearly above all others (green, at the top). This is not a problem: a higher gain amplitude means this antenna *recorded weaker raw data* for the same sky, and calibration scales it up. It is only a concern if it changes strongly with time, and it does not.

{% include fig.html src="07_gain_cal_G1_fluxcal_J1939-6342/G1_J1939-6342_phase_vs_time_all_antennas.png" caption="Gain phases on J1939-6342, all antennas." %}

{% include gallery.html match="G1_J1939-6342_phase_vs_time_per_antenna" title="Show phase plots for all antennas" %}

The phases cover the whole range from −180° to +180°. Again, this is normal: only the change of each antenna with time matters. In the per-antenna plots, each antenna is steady.

### PKS1313-333 (gain calibrator)

{% include fig.html src="08_gain_cal_G1_phasecal_PKS1313-333/G1_PKS1313-333_amp_vs_time_all_antennas.png" caption="Gain amplitudes on PKS1313-333, all antennas, over the whole night." %}

{% include fig.html src="08_gain_cal_G1_phasecal_PKS1313-333/G1_PKS1313-333_amp_vs_time_per_antenna_m000-m008.png" caption="The same, per antenna (m000 to m008)." %}

{% include gallery.html match="G1_PKS1313-333_amp_vs_time_per_antenna" title="Show amplitude plots for all antennas" %}

**What we see:**

- One solution every ~15 minutes, from 01:45 to 12:30 UTC.
- The amplitudes (about 0.45 to 0.85) are **lower than on J1939-6342**. This is the "1 Jy model" effect: PKS1313-333 is fainter than 1 Jy, so its solutions came out lower. Step 9 corrects this.
- Each antenna changes smoothly and slowly, by a few percent over the night. Some antennas (for example m000) drop a little at the very end, when the source sets lower in the sky. This is the kind of change gain calibration is designed to follow.
- The gaps in time (around 05:50, 08:20 and 10:45) are the J1939-6342 visits.

{% include fig.html src="08_gain_cal_G1_phasecal_PKS1313-333/G1_PKS1313-333_phase_vs_time_all_antennas.png" caption="Gain phases on PKS1313-333, all antennas." %}

{% include fig.html src="08_gain_cal_G1_phasecal_PKS1313-333/G1_PKS1313-333_phase_vs_time_per_antenna_m000-m008.png" caption="The same, per antenna (m000 to m008)." %}

{% include gallery.html match="G1_PKS1313-333_phase_vs_time_per_antenna" title="Show phase plots for all antennas" %}

**What we see:** each antenna's phase drifts **smoothly**, typically by 50–100° over 11 hours, often in a "U" shape (a slow change that reverses near the middle of the night, when the source is highest). There are only a few single outlier points. **m008** looks like it jumps between −180° and +180°, but this is just **phase wrapping**: its phase sits near ±180°, which are the same angle.

{: .ours }
> Smooth phase changes of a few degrees per scan mean that interpolating between calibrator scans (every ~15 minutes) works well. The ionosphere was quiet during this observation.

### J1311-2216 (spare gain calibrator)

{% include fig.html src="09_gain_cal_G1_altcal_J1311-2216/G1_J1311-2216_amp_vs_time_all_antennas.png" caption="Gain amplitudes on J1311-2216, all antennas." %}

{% include fig.html src="09_gain_cal_G1_altcal_J1311-2216/G1_J1311-2216_phase_vs_time_all_antennas.png" caption="Gain phases on J1311-2216, all antennas." %}

{% include gallery.html match="G1_J1311-2216_amp_vs_time_per_antenna" title="Show amplitude plots for all antennas" %}

{% include gallery.html match="G1_J1311-2216_phase_vs_time_per_antenna" title="Show phase plots for all antennas" %}

The amplitudes here (about 1.25 to 2.35) are *higher* than on the other two sources. With a 1 Jy model, the solved amplitude grows with the square root of the source's true flux, so this tells us J1311-2216 is **brighter** than 1 Jy. Comparing with J1939-6342 gives roughly (1.5 / 0.7)² × 1 Jy ≈ 4–5 Jy. (In the raw overview plot of step 2, J1311-2216 was also about seven times brighter than PKS1313-333.) The behaviour with time is equally smooth. We solve for this source only to have a **backup**: if PKS1313-333 had problems, we could use J1311-2216 instead. In this reduction we do not apply it (the notes also say "not done yet" for it).

## What can go wrong

{: .problem }
> **Gain calibrator too far from the target.** The atmosphere in its direction differs from the target's direction. The phases do not transfer well, and the image has errors.
>
> **Cure:** choose a calibrator within a few degrees (PKS1313-333 is about 4° from M83). Self-calibration on the target can fix what remains.

{: .problem }
> **Too few calibrator visits.** If the phase changes a lot between two visits, interpolation cannot follow it.
>
> **How to notice:** phase jumps of more than ~30–40° between neighbouring calibrator scans.
>
> **Cure:** for this data, nothing (the observation is done). For a new proposal, visit the calibrator more often. Self-calibration can help.

{: .problem }
> **A bad solution in one scan.** One calibrator scan with RFI or a glitch gives a wrong gain. This wrong gain is interpolated onto the target scans on both sides of it.
>
> **How to notice:** single outlier points in the gain plots.
>
> **Cure:** flag that calibrator scan (for that antenna or all antennas) and solve again. Interpolation then uses its neighbours.

{: .problem }
> **Forgot `append=True`.** Each `gaincal` call overwrites the table, and only the last source remains. Step 9 then fails.

{: .problem }
> **Solving the gain calibrator with a bad flux model.** If you set a model for PKS1313-333 that is wrong (for example from an old catalogue), the amplitudes are biased.
>
> **Cure:** do as we did: solve against 1 Jy, then use `fluxscale` (step 9).
