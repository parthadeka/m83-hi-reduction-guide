---
title: "12. Continuum subtraction"
parent: Reduction steps
nav_order: 12
permalink: /steps/12-continuum-subtraction/
---

# Step 12. Continuum subtraction
{: .no_toc }

Pipeline stage: `contsub` · CASA task: `uvcontsub`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

The sky at 1.4 GHz contains two kinds of radio emission:

- **Continuum emission**: broad emission at *all* frequencies, from things like hot electrons in magnetic fields (synchrotron emission). M83 itself is a bright continuum source, and the field contains hundreds of background radio galaxies. Continuum emission changes only slowly and smoothly with frequency.
- **Line emission**: the HI line, which appears only in a narrow range of channels.

For HI work, the continuum is in the way. In our data, M83's continuum (about 2 Jy in total) is far brighter than its HI in any single channel. **Continuum subtraction** removes the continuum and leaves only the line.

## How it works

For **each baseline** (and each time and polarisation), `uvcontsub` takes the spectrum, and:

1. looks only at the **line-free channels** (`ref_chans`: 1250–1420 and 1580–1800);
2. fits a smooth curve (here a polynomial of order 3) through them;
3. subtracts that curve from **all** channels, including the line channels.

In the line-free channels, only noise remains. In the line channels, the HI line remains.

{: .plain }
> Imagine a long, gently sloping road with a small hill on it. The road is the continuum, the hill is the HI line. We measure the road before and after the hill, draw the road under the hill, and then measure only the height of the hill above the road.

Working on the visibilities (not on images) is called **uv-plane continuum subtraction**. It works best for continuum sources near the centre of each pointing.

## What we ran

```python
uvcontsub(vis=vis_hi, outputvis=vis_hi_sub, fitspec='0:1250~1420;1580~1800',
          fitorder=3, datacolumn='data', writemodel=True)
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `fitspec` | channels 1250–1420 and 1580–1800 | The line-free channels from the notes. M83's HI is at about channels 1488–1538. The 70-channel gaps on both sides (about 380 km/s) make sure that no faint HI from the outer disk enters the fit. |
| `fitorder` | `3` | A cubic polynomial. It can follow a curved continuum (from the spectral index, the bandpass, and sources far from the field centre, whose spectra ripple). A higher order can follow the data too closely, and starts to fit noise or even the line. A lower order (0 or 1) may leave curved residuals. Order 3 is a common compromise for ~14 MHz of bandwidth; the notes used it too. |
| `datacolumn` | `'data'` | In the split target MS, the calibrated data are in `DATA`. |
| `writemodel` | `True` | Store the fitted continuum in the `MODEL_DATA` column of the new MS. We use it for the "continuum model" plot below. It is kept for inspection. |
| `outputvis` | a new MS (`.contsub`) | CASA 6's `uvcontsub` writes the result to a new MS. The original target MS is not changed, so it can still be used for continuum imaging (step 16) and for the cube without continuum subtraction. |

{: .note }
> In CASA 5 (as used in the notes), the task was called with `fitspw=` and `want_cont=True`, and it wrote a `.contsub` MS next to the input. CASA 6.5 and later have a new `uvcontsub` with `fitspec=` and `outputvis=`. The idea is the same.

## The plots

All the per-field plots below are averaged over all times and all baselines. As explained before, this keeps only the emission at the **centre** of each pointing.

### Before subtraction

{% include fig.html src="12_continuum_subtraction/before_contsub_amp_vs_chan_per_field.png" caption="Before continuum subtraction: amplitude against channel, per pointing." %}

{% include fig.html src="12_continuum_subtraction/before_contsub_real_vs_chan_per_field.png" caption="Before continuum subtraction: real part of the visibility against channel, per pointing." %}

The **real part** is the part of the complex visibility that lines up with the phase centre. Unlike the amplitude, it can be **negative**, so it shows whether something adds to or takes away from the signal at the field centre.

**What we see:** M83_4 sits at about **0.2 Jy**: the continuum of M83's centre. The two polarisations differ by about 2%. The other pointings are close to zero, with only small offsets of a few mJy from continuum sources near their centres.

### The fitted continuum

{% include fig.html src="12_continuum_subtraction/continuum_model_real_vs_chan_per_field.png" caption="The continuum fitted by uvcontsub (MODEL_DATA, real part), per pointing." %}

**What we see:** smooth curves, as expected from a cubic polynomial. For M83_4 it is a gentle straight slope from 0.205 to 0.199 Jy: M83's continuum getting fainter with frequency. In the other pointings, the curves are very small (below 2 mJy) and gently curved. Note the y-axis ranges; the curves look dramatic, but they are tiny.

### After subtraction

{% include fig.html src="12_continuum_subtraction/after_contsub_real_vs_chan_per_field.png" caption="After continuum subtraction: real part against channel, per pointing." %}

{% include fig.html src="12_continuum_subtraction/after_contsub_amp_vs_chan_per_field.png" caption="After continuum subtraction: amplitude against channel, per pointing." %}

**What we see:**

- **All pointings are now centred on zero** in the line-free channels, with no slopes or curves left. The continuum has been removed well.
- **M83_4 shows a negative feature** (in the real part) between channels about 1490 and 1540, down to about −6 mJy. In the amplitude plot it appears as a positive peak, because amplitude is always positive. **This is M83's HI**, at M83's velocity.

**Why is the HI negative here?** There are two possible reasons, and this plot alone cannot separate them:

1. **HI absorption.** Cold HI gas in front of M83's bright nucleus can absorb some of the nucleus's continuum, just like the Milky Way absorption we saw on the calibrators.
2. **A "negative bowl".** M83's HI is very extended. The average over all baselines measures the **dirty image** at the centre pixel. When a large, smooth structure is not fully captured by the shortest baselines, the dirty image has a negative dip around and under it.

The image cube (step 14) will tell us which it is: a spectrum at the exact position of the nucleus will show whether there is real absorption.

- **A narrow feature near channel ~1600–1610** in M83_4 and M83_5. This is **Milky Way HI** at about 0 km/s (1420.4 MHz). It is outside M83's line, so it does not affect M83. But it is *inside* the continuum-fit range (1580–1800), so it pulls the polynomial very slightly. With only ~10 affected channels among ~390, the effect is very small.

### After subtraction, per antenna

{% include fig.html src="12_continuum_subtraction/after_contsub_amp_vs_chan_per_antenna_m000-m006.png" caption="After continuum subtraction: amplitude against channel per antenna (m000 to m006), averaged over time. Colours are baselines." %}

{% include gallery.html match="12_continuum_subtraction/after_contsub_amp_vs_chan_per_antenna" %}

Here we do not average over baselines, so we see the HI from many baselines. The line-free channels are **flat and low** on every antenna. **M83's HI** stands out clearly between channels 1490 and 1540, reaching 1–2 Jy on the shortest baselines. A small group of points near **channel 1600–1610** is again the **Milky Way HI**.

## What can go wrong

{: .problem }
> **Line channels inside the fit range.** If `fitspec` includes channels with HI, the fit includes the line, and part of the line is subtracted.
>
> **How to notice:** the HI is weaker than expected; there are negative "wings" next to the line.
>
> **Cure:** choose the line-free channels from a first quick cube (or from known velocities), with generous margins. Our margins are about 70 channels on each side.

{: .problem }
> **Fit order too high.** A high-order polynomial follows the noise, and can bend into the line region, removing faint emission.
>
> **Cure:** use the lowest order that leaves flat residuals. Order 1–3 is usual.

{: .problem }
> **Fit order too low, or bright sources far from the field centre.** A source far from the phase centre has a visibility phase that changes with frequency (it "ripples" along the band). A low-order polynomial cannot follow these ripples, and the residual continuum leaves stripes in the cube.
>
> **How to notice:** in the cube, line-free channels show the shape of bright continuum sources, or there are ripples.
>
> **Cure:** a slightly higher order; or subtract a **continuum model** first (image the continuum, subtract the model from the visibilities with `uvsub`, then run `uvcontsub` on what is left); or remove the remaining continuum in the image domain afterwards (`imcontsub`).

{: .problem }
> **Other spectral lines in the fit range.** Here, Milky Way HI at channel ~1600.
>
> **Cure:** exclude those channels from `fitspec` too, for example `'0:1250~1420;1580~1590;1615~1800'`.

{: .problem }
> **Continuum subtraction before the calibration is final.** Self-calibration later would require doing this step again.
>
> **Cure:** finish all calibration (including self-calibration, if needed) first.
