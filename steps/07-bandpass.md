---
title: "7. Bandpass calibration"
parent: Reduction steps
nav_order: 7
permalink: /steps/07-bandpass/
---

# Step 7. Bandpass calibration (B)
{: .no_toc }

Pipeline stage: `bandpass` · CASA task: `bandpass`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

No receiver is equally sensitive at all frequencies. Filters, amplifiers and cables make the response go up and down a little across the band. This response, as a function of frequency, is called the **bandpass**.

This step measures the bandpass of every antenna and polarisation, **channel by channel**: one amplitude and one phase for each of the 2001 channels in our MS. (Unlike steps 6 and 8, the bandpass call does not select only the good channels, so it covers the whole 1378–1431 MHz range.)

## Why we need it

For HI work the bandpass is especially important. We are looking for a weak spectral line. If the receiver has a small bump at some frequency, that bump looks exactly like a weak line. If the receiver has a slope, our continuum subtraction (step 12) becomes harder. A correct bandpass makes the spectrum of every source flat and true.

## Why J1939-6342?

A good bandpass calibrator must be:

- **bright**, so that each single channel has enough signal (we solve 2001 channels separately, so each channel only has a tiny part of the total signal);
- **without spectral lines** of its own (or with those channels flagged; we flagged the Milky Way HI absorption in step 3);
- **with a smooth, known spectrum**.

J1939-6342 (15 Jy) is one of the best bandpass calibrators in the southern sky.

## What we ran

```python
bandpass(vis=msfile, caltable=btable0, field='J1939-6342', refant='m059',
         solnorm=True, combine='scan', solint='inf', bandtype='B', minsnr=5,
         gaintable=[ktable0, gtable0], interp=['nearest', 'nearest'])
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `bandtype` | `'B'` | Solve each channel independently. (The other option, `'BPOLY'`, fits a smooth polynomial. It is less noisy, but cannot follow narrow features.) |
| `solint='inf'`, `combine='scan'` | one solution from all three scans | The bandpass changes very slowly. Combining the scans gives the highest SNR per channel. |
| `gaintable` | `[ktable0, gtable0]` | Apply the delays (step 5) and the phase drift between scans (step 6) first. This is why we needed G0. |
| `solnorm` | `True` | Scale the bandpass so that its average amplitude is 1. The bandpass then only describes the *shape* across frequency. The overall amplification is measured later, in step 8. |
| `minsnr` | `5` | Reject channels with too little signal. |

## The plots

### Bandpass amplitude

{% include fig.html src="06_bandpass_cal_B0/bpass_amp_per_antenna_m000-m008.png" caption="Bandpass amplitude against frequency for antennas m000 to m008. Black and pink are the two polarisations." %}

{% include gallery.html match="06_bandpass_cal_B0/bpass_amp" %}

**How to read it.** The y-axis is the bandpass amplitude. Because of `solnorm=True`, it is close to 1. A value of 1.04 means the receiver is 4% more sensitive than average at that frequency.

**What we see:**

- Most antennas have a **gentle slope**: about +4% at the low-frequency end and −4% at the high end. This is the normal shape of the MeerKAT L-band receiver in this part of the band.
- **The curves are smooth**, with no narrow spikes. This means there is no RFI or line left in the calibrator data.
- **m001 has a ripple** in one polarisation (black): a smooth wave of about ±2–3% with a period of about 25 MHz. This is a **standing wave**, caused by signal reflections inside that receiver chain. It does not matter, because the bandpass correction removes it from the data. It is useful to know about, because a ripple that is *not* corrected would look like a broad line.
- **m003 is empty** (flagged).
- The solutions cover the whole MS, 1378–1431 MHz. Only the middle part, 1411–1425 MHz, is used for the science.

### Bandpass phase

{% include fig.html src="06_bandpass_cal_B0/bpass_phase_per_antenna_m000-m008.png" caption="Bandpass phase against frequency for antennas m000 to m008." %}

{% include gallery.html match="06_bandpass_cal_B0/bpass_phase" %}

**What we see:** the bandpass phases are all within about **±1°** (m001's ripple again reaches ±1.5°). There is no remaining slope, which confirms that the delay calibration (step 5) worked. Small, smooth phase changes like these are normal.

{: .note }
> The script also tries to make four "all antennas in one plot" bandpass plots (`bpass_amp_x_all_antennas` and so on). They failed in this run, because the first version of the script asked plotms for polarisation `'XX'` instead of `'X'` on a calibration table. The script is fixed now; rerunning only this stage (`M83_STAGES=bandpass`) will make them.

## What can go wrong

{: .problem }
> **Spikes in the bandpass.** A narrow spike in amplitude or phase usually means RFI or a spectral line in the calibrator data.
>
> **Why it matters:** the spike is divided into every target spectrum. It looks like a false line in the final cube.
>
> **Cure:** flag those channels in the calibrator data, and solve again.

{: .problem }
> **Noisy bandpass.** The solutions scatter a lot from channel to channel.
>
> **Cause:** not enough signal per channel (calibrator too faint, too little time).
>
> **Cure:** combine more scans; or average a few channels in the solution (`solint='inf,4ch'` gives one solution per 4 channels); or use a polynomial (`bandtype='BPOLY'`). Bandpass noise adds noise to every target spectrum.

{: .problem }
> **Bandpass changes during the night.** If you solve per scan and the shapes differ, one combined bandpass will not fit all times.
>
> **Cure:** usually the bandpass is stable for hours. If not, solve per scan and apply with `interp='linear'` in time.

{: .problem }
> **Forgot to apply the delay or initial phase table.** Phase drifts between scans make the combined solution partly cancel, so the bandpass amplitude comes out too low and noisy, and the phase keeps a slope.
>
> **Cure:** always list all earlier tables in `gaintable`.
