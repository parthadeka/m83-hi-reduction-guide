---
title: "6. Initial phase calibration"
parent: Reduction steps
nav_order: 6
permalink: /steps/06-initial-phase/
---

# Step 6. Initial phase calibration (G0)
{: .no_toc }

Pipeline stage: `gain0` · CASA task: `gaincal` with `calmode='p'`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

This step finds one **phase** correction per antenna, per polarisation, per scan of J1939-6342. It does not change amplitudes (`calmode='p'` means "phase only").

## Why we need it

This table is a **helper**. It is only used in the next step, bandpass calibration, and is then thrown away. It is never applied to the target.

Here is why we need it. The bandpass task combines the three J1939-6342 scans (at 05:50, 08:20 and 10:45) to get one bandpass. But the phase of each antenna drifts between those scans, because the atmosphere and electronics change. If we averaged the scans without correcting this drift first, the phases would partly cancel, and the bandpass would be wrong. G0 removes the drift, so the three scans line up before they are averaged.

{: .plain }
> Imagine averaging three photos of the same object, taken while the camera was moving a little. First you must line the photos up. G0 lines up the three calibrator scans.

## What we ran

```python
gaincal(vis=msfile, field='J1939-6342', caltable=gtable0, refant='m059',
        spw='0:1250~1800', calmode='p', solint='inf', minsnr=5, solnorm=True,
        gaintable=[ktable0], interp='nearest')
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `calmode` | `'p'` | Phase only. Amplitudes are solved properly later (step 8). |
| `solint` | `'inf'` | One solution per scan (no `combine`), so we follow the drift between scans. |
| `spw` | `'0:1250~1800'` | Use only the good part of the band (`valid_chans`). The band edges are noisier. |
| `gaintable` | `[ktable0]` | Apply the delays from step 5 **on the fly** while solving. Each new calibration table is always solved "on top of" the earlier ones. |
| `interp` | `'nearest'` | How to use the earlier table at times where it has no solution. The delay table has one value for the whole night, so "nearest" simply uses that value. |
| `solnorm` | `True` | Normalise the solutions. For phase-only solutions this has almost no effect; it is kept from the notes. |
| `minsnr` | `5` | As in step 5. |

## The plots

{% include fig.html src="05_initial_phase_cal_G0/G0_phase_vs_time_per_antenna_m000-m008.png" caption="G0 phase solutions against time, for antennas m000 to m008. Each point is one J1939-6342 scan; black and pink are the two polarisations." %}

{% include gallery.html match="05_initial_phase_cal_G0/G0_phase" %}

**How to read it.** Each panel is one antenna. The y-axis is the phase correction, in degrees. Each antenna has one point per scan and polarisation.

**What we see:**

- The phases are anywhere between −180° and +180°. **That is normal.** The absolute value of a phase has no meaning; only its *change* matters.
- For each antenna, the three scans give similar phases. They change by only **10–30°** over five hours. This means the atmosphere and electronics were stable.
- The two polarisations have different phases. That is also normal: the X and Y receivers have separate electronics.
- m003 is empty (flagged). m004 has only two scans (its first scan was flagged).

## What can go wrong

{: .problem }
> **Phases jump by large amounts between scans** (for example more than 90°), or jump around randomly.
>
> **How to notice:** in the plot, the points of one antenna do not follow a smooth line.
>
> **Cure:** smooth jumps are fine (G0 is designed to correct them). Random jumps mean low signal or a bad antenna. Check that antenna in the raw data, and flag it if needed.

{: .problem }
> **Phase wrapping looks like a jump.** A phase of +179° and one of −179° are only 2° apart, but in the plot they appear at opposite edges.
>
> **Cure:** nothing to fix. Read such points as close together.

{: .problem }
> **Applying G0 to the target by mistake.** G0 is measured only at three times, on a source far from M83 on the sky. It does not describe how the gains change during the target scans.
>
> **Cure:** use G0 only inside the bandpass step. The final calibration of the target uses G1 (step 8) instead.
