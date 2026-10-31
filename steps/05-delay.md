---
title: "5. Delay calibration"
parent: Reduction steps
nav_order: 5
permalink: /steps/05-delay/
---

# Step 5. Delay calibration (K)
{: .no_toc }

Pipeline stage: `delay` · CASA task: `gaincal` with `gaintype='K'`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

A **delay** is a small timing error: the signal from one antenna arrives at the correlator a tiny bit earlier or later than it should. The cause can be cable lengths, electronics, or small errors in the antenna positions.

A time delay τ changes the phase by 360° × frequency × τ. So a delay makes the phase **change linearly with frequency** (a straight sloped line in a phase-against-frequency plot). If we do not correct it, the phases at the two ends of the band disagree, and averaging over frequency makes the signal partly cancel. We say the signal **decorrelates**.

This step measures one delay per antenna and per polarisation, in **nanoseconds** (ns, 10<sup>−9</sup> s).

## Why we need it

The next steps (initial phase and bandpass) average over frequency or fit across the band. They work much better when the phase slope has already been removed.

## The reference antenna

Phases are **relative**: we can only measure the *difference* between two antennas. So we choose one antenna, the **reference antenna** (`refant`), and set its phase (and delay) to zero. All other antennas are measured relative to it.

A good reference antenna:

- has no flagged data and no problems during the whole observation;
- is stable;
- is often near the centre of the array (but this is not necessary).

The notes chose **`m059`**, and we kept it. It was stable in all the raw plots.

## What we ran

We ran `gaincal` twice:

```python
# 1. one solution per scan (for checking only)
gaincal(vis=msfile, caltable=ktable0_scan, field='J1939-6342', gaintype='K',
        solint='inf', minsnr=5.0, refant='m059')

# 2. one solution for the whole observation (this one is used)
gaincal(vis=msfile, caltable=ktable0, field='J1939-6342', gaintype='K',
        solint='inf', combine='scan,field', minsnr=5.0, refant='m059')
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `field` | `'J1939-6342'` | A bright source gives a precise delay. |
| `gaintype` | `'K'` | Solve for delays. |
| `solint` | `'inf'` | **Solution interval**: how much time goes into one solution. `'inf'` means "as long as possible". Without `combine`, a solution never crosses a scan boundary, so we get one per scan. |
| `combine` | `'scan,field'` | Let the solution cross scan boundaries: one solution for all three J1939-6342 scans together. Delays hardly change during a night, and more data gives a more precise answer. |
| `minsnr` | `5` | Reject solutions with a **signal-to-noise ratio** below 5. A low-SNR solution is mostly noise. |
| `refant` | `'m059'` | See above. |

The first table (per scan) is only for checking that the delays are stable. The second table (combined) is the one we apply.

## The plots

{% include fig.html src="04_delay_cal_K0/K0_delay_vs_antenna.png" caption="The delay used for calibration (combined over all scans), for each antenna. Black and pink are the two polarisations." %}

**What we see:** all delays are **between −0.12 and +0.48 ns**. That is very small. MeerKAT already corrects most delays at the telescope, so we only measure what is left over. A delay of 0.5 ns makes a phase slope of only 360° × 14 MHz × 0.5 ns ≈ 2.5° across our 14 MHz band. The x-axis is the antenna *index* (counting from 0), not the antenna name. Four names are missing (m002, m021, m048, m052), so the reference antenna m059 has index 55. It sits at 0 ns, as expected.

{% include fig.html src="04_delay_cal_K0/K0_perscan_delay_vs_time_per_antenna_m000-m008.png" caption="Delay per scan for antennas m000 to m008. Each point is one J1939-6342 scan." %}

{% include gallery.html match="04_delay_cal_K0/K0_perscan" %}

**What we see:**

- For each antenna, the three scans give almost the same delay (within about 0.05 ns). The delays are stable over the night, so combining all scans into one solution is safe.
- **m003 has no points**: it is fully flagged.
- **m004 has only two points** (at 08:20 and 10:45). Its first scan at 05:50 was flagged, most likely by the shadow flagging in step 3.

## What can go wrong

{: .problem }
> **Large delays (tens of nanoseconds or more).** Usually a sign of a problem at the telescope, or of a wrong antenna position.
>
> **How to notice:** the delay-vs-antenna plot shows one antenna far from the others; its phase against frequency "wraps" many times across the band.
>
> **Cure:** a large but *stable* delay is fine; calibration removes it. If the delay *jumps* between scans, solve per scan (no `combine`) or flag the antenna.

{: .problem }
> **Bad reference antenna.** If the reference antenna has missing data for some time, solutions for all antennas fail during that time.
>
> **How to notice:** many antennas lose solutions at the same times.
>
> **Cure:** choose another reference antenna. CASA also accepts a list (`refant='m059,m000,m010'`) and uses the next one when the first is missing.

{: .problem }
> **Solutions rejected for low SNR.** CASA prints "solutions failed (SNR < minsnr)".
>
> **Cure:** use a brighter calibrator, a longer `solint`, or `combine='scan'`. Lowering `minsnr` is possible but lets noisy solutions in.

{: .note }
> CASA reports delays in nanoseconds for `gaintype='K'`. They are "residual" delays: the large geometric delays (from the source position and antenna positions) were already removed by the correlator.
