---
title: "10. Apply and check the calibration"
parent: Reduction steps
nav_order: 10
permalink: /steps/10-apply-and-check/
---

# Step 10. Apply the calibration and check it
{: .no_toc }

Pipeline stages: `applycal`, `inspect_corrected` · CASA tasks: `applycal`, `plotms`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

Until now we have only *measured* the corrections and stored them in tables. Now we **apply** them: CASA divides the raw data by the gains and writes the result into a new column of the MS, `CORRECTED_DATA`. The raw `DATA` column is not changed.

Then we plot the corrected data to check that the calibration worked.

## What we ran

```python
tables = [ktable0, btable0, ftable1]
interp = ['nearest', 'nearest', 'linear']

applycal(vis=msfile, field='M83_0,...,M83_6', gaintable=tables,
         gainfield=['J1939-6342', 'J1939-6342', 'PKS1313-333'], interp=interp)
applycal(vis=msfile, field='PKS1313-333', gaintable=tables,
         gainfield=['J1939-6342', 'J1939-6342', 'PKS1313-333'], interp=interp)
applycal(vis=msfile, field='J1939-6342', gaintable=tables,
         gainfield=['J1939-6342', 'J1939-6342', 'J1939-6342'], interp=interp)
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `gaintable` | K, B, fluxscale1 | The delay, the bandpass, and the flux-scaled gains. Not G0 (it was only a helper for the bandpass) and not G1 (replaced by `fluxscale1`). |
| `gainfield` | one per table | Which source's solutions to use from each table. For the target: delay and bandpass from J1939-6342, time-variable gains from PKS1313-333. |
| `interp` | nearest, nearest, **linear** | Delay and bandpass have one solution for the night, so "nearest" just uses it. The gains change with time, so for each target time we **draw a straight line** between the PKS1313-333 scan before and the one after (linear interpolation). |

We also apply the calibration to the two calibrators themselves. This is only for checking: a calibrator corrected with its own solutions must look like its model.

It took about **90 minutes**, mainly reading and writing the large MS.

## Checking the flux calibrator

{% include fig.html src="10_corrected_data_inspection/fluxcal_corrected_amp_vs_freq.png" caption="J1939-6342 after calibration: amplitude against frequency, averaged over all times and baselines. The colours are XX and YY." %}

**What we see:**

- The amplitude is **14.85 to 14.89 Jy**, sloping gently down with frequency. This is exactly the Stevens-Reynolds model from step 4 (14.95 Jy, with its spectral index). The calibration reproduces the model.
- **XX and YY lie on top of each other.** The difference between the X and Y receivers is gone.
- **The gap near 1.4203 GHz** is the channels we flagged in step 3 (Milky Way HI). A few points at the edges of the gap dip slightly; those channels still contain a little of the absorption line. It does not matter, because J1939-6342 is only used for calibration, and those channels lie outside M83's line.

## Checking the gain calibrator

{% include fig.html src="10_corrected_data_inspection/phasecal_corrected_phase_vs_time.png" caption="PKS1313-333 after calibration: phase against time." %}

The phases are now **centred on 0°** at all times. Before calibration they were spread over ±180° and drifted. The remaining scatter (about ±50° for individual visibilities) is **noise**: PKS1313-333 is only 0.72 Jy, and each point is one baseline over one 8-second dump. A few points near 06:50 reach ±100°; these are single noisy visibilities.

{% include fig.html src="10_corrected_data_inspection/phasecal_corrected_amp_vs_time.png" caption="PKS1313-333 after calibration: amplitude against time." %}

The amplitudes are the same in every scan through the night. There are no jumps and no slow trends. The calibration followed the gain changes correctly.

{% include fig.html src="10_corrected_data_inspection/phasecal_corrected_amp_vs_uvdist.png" caption="PKS1313-333 after calibration: amplitude against baseline length (uv distance, in metres)." %}

**How to read it.** A perfect point source at the field centre gives the **same amplitude on every baseline**, so the points would form a flat line.

**What we see:** on long baselines (above about 4 km) the amplitude settles at about **0.7 Jy**, which is the fluxscale result. On short baselines it is higher and more spread out, up to about 1.2 Jy. This is because **short baselines also see other, fainter sources** in the 1° primary beam around PKS1313-333, and possibly some extended emission. Long baselines are only sensitive to small, compact things, so they see PKS1313-333 alone. This is normal for MeerKAT, which is so sensitive that every field contains many background sources.

{: .note }
> This is why we solved the gains with `solint='inf'` per scan, over all baselines: the extra sources average out over a whole scan. If they were very bright, we would need a model of the whole field, or a `uvrange` limit to long baselines when solving.

## Checking the target

### Amplitude against channel

{% include fig.html src="10_corrected_data_inspection/targets_corrected_amp_vs_chan_per_field.png" caption="Corrected data of each M83 pointing, averaged over time and all baselines. Colours: XX and YY." %}

All pointings are flat across the band, with no spikes. **M83_4** (centred on the galaxy) shows M83's continuum at about 0.2 Jy, plus a small **dip** between channels 1490 and 1540: this is where **M83's HI line** is. (Why a dip and not a bump? See [step 12](../12-continuum-subtraction/).) The other pointings look like noise. As explained in [step 2](../02-first-look/), averaging over all baselines only keeps what is at the centre of each pointing.

{% include fig.html src="10_corrected_data_inspection/targets_corrected_amp_vs_chan_per_antenna_m000-m006.png" caption="Corrected target data, per antenna (m000 to m006), averaged over time. Colours are baselines." %}

{% include gallery.html match="10_corrected_data_inspection/targets_corrected_amp_vs_chan_per_antenna" %}

Here each colour is one baseline, and the curves are not averaged over baselines. Now M83's HI line is clearly visible as a **bump around channels 1490–1540** in many baselines. It is strongest on the **shortest baselines**. In each panel, one curve sits higher than the rest, at about 2.2 Jy: this is a very short baseline, which sees almost all of M83's bright, extended continuum emission. Longer baselines **resolve** (break up) M83's large structure, so they see less of it.

This tells us something important for later: **M83's HI is very extended, and the short baselines carry most of its signal.**

### Amplitude against time: a spike

{% include fig.html src="10_corrected_data_inspection/targets_corrected_amp_vs_time.png" caption="Corrected data of all M83 pointings against time, averaged over channels 1250–1800. Colours are fields." %}

Almost all the data are between 0 and about 3 Jy. But there is **one narrow spike** around **07:02–07:05 UTC**, in one colour (one field), reaching **about 54 Jy**.

**What could it be?** Comparing the time with the observation log:

| Scan | Time (UTC) | Field |
|:-----|:-----------|:------|
| 155 | 07:00:30 – 07:02:06 | M83_5 |
| 156 | 07:02:30 – 07:04:06 | M83_6 |
| 157 | 07:05:10 – 07:06:06 | J1311-2216 (calibrator, not in this plot) |

So the spike is very probably in **scan 156 (M83_6)**, or possibly at the end of scan 155. It is very short in time. A bad gain solution would affect *all* target scans between two calibrator visits (about 15 minutes), not just one. So the most likely cause is a **short burst of RFI or a correlator/digitiser glitch** on one or a few antennas.

{: .ours }
> This spike is the one real data problem we found in this dataset. It was not removed before the next steps. The noise we measured in the images (step 13) agrees with the theoretical value, so its effect seems small. The proper fix is to find the affected antenna(s) and time, flag them, and image again.

**How to find and fix it:**

1. Plot only that time range (`timerange='06:58:00~07:05:00'`), with amplitude against antenna (`xaxis='antenna1'`) for the `corrected` and the raw `data` column. If it is in the raw data too, it is a data problem, not a calibration problem.
2. Flag those antennas and times with `flagdata(mode='manual', timerange=..., antenna=...)`. Because the target MS is split off in step 11, the same flags must be applied to the target MS and the continuum-subtracted MS.
3. Make the images again (steps 13 onward). The long calibration steps do not need to be repeated.

## What can go wrong

{: .problem }
> **Calibrator does not look like its model** (wrong flux, slope, or remaining bumps).
>
> **Cure:** go back to the step that went wrong: flux model (step 4), bandpass (step 7), or gains (step 8).

{: .problem }
> **Phases of the calibrated gain calibrator are not centred on 0°**, or drift with time.
>
> **Cause:** wrong `gainfield` or `interp`, or a bad solution in G1.
>
> **Cure:** check the `applycal` call; flag bad calibrator scans.

{: .problem }
> **Spikes or outliers in the target data** (like ours).
>
> **Cure:** find them (time, antenna, channel), flag them, and reprocess from that point.

{: .problem }
> **Wrong interpolation.** `interp='nearest'` for the gains would give a "staircase" (the gain jumps at the middle between calibrator scans). `'linear'` follows slow changes better.

{: .problem }
> **Applying to the wrong fields.** `applycal` only writes `CORRECTED_DATA` for the fields you list. Fields left out keep an uncorrected (or no) `CORRECTED_DATA`.
