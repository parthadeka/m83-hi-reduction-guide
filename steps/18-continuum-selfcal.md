---
title: "18. Continuum self-calibration"
parent: Reduction steps
nav_order: 18
permalink: /steps/18-continuum-selfcal/
---

# Step 18. Better continuum: self-calibration and a new mosaic
{: .no_toc }

Scripts: `m83_cont_split.py`, `run_m83_selfcal.slurm` + `m83_selfcal_gain.py`, `run_m83_cont_final.slurm` + `m83_cont_final_mask.py`, `m83_cont_mosaic.py` · Software: CASA (`split`, `gaincal`, `applycal`), WSClean, katbeam
{: .step-meta }

1. TOC
{:toc}

---

## Why this step

The continuum image of [step 16](../16-continuum-image/) had a noise of **0.22 mJy/beam**, about 6 times the thermal noise. It showed a negative bowl around M83, rings around bright sources, and faint patches everywhere. Its clean also stalled: the remaining errors were calibration errors, which cleaning cannot remove.

This step fixes the calibration with **self-calibration**, and cleans better. The result is a mosaic with **0.038 mJy/beam** in the centre: **almost 6 times deeper**, and close to the thermal limit. This step is not in the notes.

## What is self-calibration?

In steps 5 to 10 we measured the antenna gains on calibrator sources, and copied them to M83 by interpolation between calibrator visits. This is never perfect: the atmosphere in the direction of M83 is a little different, and the gains change between visits.

**Self-calibration** ("selfcal") uses the target field itself as the calibrator:

1. Make the best image you can, and keep its **model**: the list of clean components, which is our best guess of what the sky looks like.
2. Predict, from the model, the visibilities the telescope *should* have measured.
3. Solve for the antenna gains that make the measured data agree with the prediction (`gaincal`, as in step 8, but with the model instead of a calibrator).
4. Apply the gains, and make a new, better image. Its better model gives better gains. Repeat.

{: .plain }
> It is like tuning a guitar by ear: you play a chord, hear what is slightly wrong, adjust the strings, and play again. Each round, the chord (the model) gets better, and so does the tuning (the gains).

Two kinds of selfcal are used, always in this order:

- **Phase-only** (`calmode='p'`): corrects only the phases (timing). Safe: it cannot change the brightness of sources.
- **Amplitude and phase** (`calmode='ap'`): also corrects the amplitudes. Powerful, but **dangerous**: if the model is wrong, amplitude selfcal will happily change the data to match the wrong model. Use it only when the model is good.

{: .note }
> Selfcal only works when the field contains enough bright emission to give a strong signal in each solution interval. Our field has M83 (about 2 Jy) and many background sources, so this is easy here.

## How we did it

### Each pointing on its own, with WSClean

We self-calibrated each of the 7 pointings separately, instead of the whole mosaic at once:

- each pointing has its own antenna gains at its own times, so it is natural to solve for them separately;
- **WSClean**, a fast imaging program widely used for MeerKAT, images one pointing many times faster than CASA's mosaic `tclean`. One full selfcal run of a pointing took about **30 minutes**, against about 2.5 hours for *one* major cycle of the step 16 clean;
- the 7 pointings run at the same time, as a slurm **array job** (one job per pointing).

At the end, the 7 pointings are combined into one mosaic.

### A small continuum MS per pointing (`m83_cont_split.py`)

```python
split(vis=vis_hi, outputvis=field + '_cont.ms', field=field, spw='0:1250~1800',
      width=8, datacolumn='data', keepflags=True)
flagdata(vis=field + '_cont.ms', mode='manual', spw='0:21~41')
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `spw` | `0:1250~1800` | The same channels as the HI work, including the HI channels for now |
| `width` | 8 | Average 8 channels into one (551 → 69 channels of 209 kHz). The data become 8 times smaller and faster to image. Averaging over 209 kHz smears sources very little over our field. |
| `flagdata` channels 21–41 | — | The averaged channels that contain M83's HI (input channels 1418–1585). The continuum must not include the line. These flags are only in the new MS; the original data are not changed. |

What remains: 48 channels, about **10 MHz** of line-free bandwidth.

### The selfcal loop (`run_m83_selfcal.slurm`)

For each pointing: image → solve gains → apply → image again, six images in total:

| Image | Data imaged | Then solve | Solution interval |
|:------|:------------|:-----------|:------------------|
| r0 | `DATA` (step 10 calibration only) | phase | per scan (`'inf'`) |
| r1 | `CORRECTED_DATA` | phase | 120 s |
| r2 | `CORRECTED_DATA` | phase | 60 s |
| r3 | `CORRECTED_DATA` | amplitude + phase | per scan, on top of the 60 s phases |
| r4 | `CORRECTED_DATA` | amplitude + phase | 10 min, on top of the 60 s phases |
| r5 | `CORRECTED_DATA` | (final selfcal image) | |

The **solution interval** is the length of data used for one gain solution. Short intervals follow fast changes, but each solution has less signal, so it is noisier. Each round solves from the original `DATA` against the newest model, so a bad round does not build on the one before.

```python
gaincal(vis=ms, caltable=table, solint=solint, refant='m059', calmode=calmode,
        gaintype='G', minsnr=3.0, solnorm=(calmode == 'ap'), gaintable=prior)
applycal(vis=ms, gaintable=prior + [table], interp='linear', calwt=False, applymode='calonly')
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `minsnr` | 3 | Reject solutions with a signal-to-noise below 3 |
| `solnorm` | `True` for amplitudes | Normalise the amplitude gains to an average of 1, so selfcal cannot change the overall flux scale set in step 9 |
| `gaintable` (prior) | the 60 s phases, for the amplitude rounds | Solve the amplitudes after the phases are already corrected |
| `calwt` | `False` | Do not change the data weights |
| `applymode` | `'calonly'` | Apply the gains, but **do not flag** data where a solution failed (the default would flag them, and those flags would add up over the rounds) |

The WSClean command for each image:

```
wsclean -size 5120 5120 -scale 2asec -weight briggs 0 \
        -niter 1000000 -nmiter 15 -mgain 0.8 \
        -multiscale -multiscale-scales 0,4,12,36,108 \
        -fits-mask <mask from the previous image> -auto-threshold 1 \
        -parallel-deconvolution 2560 -j 16 <field>_cont.ms
```

| Option | Value | Why |
|:-------|:------|:----|
| `-size`, `-scale` | 5120 × 2″ = 2.8° | The main beam of one pointing (about 1°) and the bright sources just outside it. 2″ pixels give about 4 pixels across the 9″ × 8″ beam. |
| `-weight briggs 0` | robust 0 | As in step 16 |
| `-nmiter` | 15 | At most 15 major cycles: a clean can never run for days again |
| `-mgain` | 0.8 | Each major cycle cleans 80% of the peak, the usual WSClean setting |
| `-multiscale-scales` | 0, 4, 12, 36, 108 px | **Multi-scale clean** uses components of several sizes: points (0) and smooth blobs of 8″ to 3.6′. Extended emission like M83's disk is described much better by a few broad components than by thousands of points. |
| `-fits-mask` | from the previous image | Clean only inside a **mask**: the pixels of the previous image above 5 × the noise, grown by 3 pixels. Round r0 has no previous image, so it uses WSClean's automatic mask (`-auto-mask 6 -local-rms`). |
| `-auto-threshold` | 1 | Inside the mask, clean down to 1 × the noise |
| `-parallel-deconvolution`, `-j` | 2560, 16 | Clean the image in 4 parts at the same time, on 16 cores |

## How we found this recipe: four tries on the central pointing

The recipe above was not our first try. We tested on the central pointing (M83_4) first. Each try taught us something, so they are all shown here.

### Try 0: WSClean chose the scales itself, and amplitude selfcal followed a bad model

Without `-multiscale-scales`, WSClean chose its own scales, up to **721 pixels (24′)**. MeerKAT has no baselines short enough to measure structures that large, so clean could put large positive and negative blobs anywhere without the data stopping it. The model filled with fake blobs.

| Image | Residual noise | Dynamic range |
|:------|---------------:|--------------:|
| r0 | 0.129 mJy | 626 |
| r1 (phase, per scan) | 0.100 mJy | 869 |
| r2 (phase, 120 s) | 0.099 mJy | 880 |
| r3 (phase, **32 s**) | 0.111 mJy | 748 |
| r4 (**amplitude** + phase) | **17.8 mJy** | **8** |

{% include fig.html src="14_continuum_selfcal/01_failed_selfcal_auto_scales.png" caption="Try 0, image r4 of M83_4 after amplitude selfcal with the bad model: smooth fake blobs of ±0.1 Jy/beam, and almost no real sources. The colour scale is set by the blobs." %}

**Dynamic range** is the peak brightness divided by the noise. It measures how well bright and faint things can be seen in one image.

{: .problem }
> **Amplitude selfcal with a bad model destroys the image.** Phase selfcal was still fine (r1, r2), because phases cannot invent brightness. Amplitude gains can: they made the data match the fake blobs.
>
> **Cure:** always set the multi-scale sizes yourself (no larger than the largest structure the array can measure); do amplitude selfcal only when the phase rounds have clearly worked; and always look at the model, not only the image.

{: .problem }
> **Too short a solution interval.** The 32 s phase round made the image *worse* (0.099 → 0.111 mJy). With only 10 MHz of bandwidth, 32 s of data per antenna does not have enough signal; the noisy solutions add noise to the data.
>
> **Cure:** use longer intervals (we changed to 60 s), and check that each round improves the image.

### Baseline: scales up to 36 px

With the scales fixed at 0, 4, 12, 36 pixels, everything worked. The image noise fell from 0.130 to 0.073 mJy, and the amplitude round gave the biggest gain. But two problems remained:

{% include fig.html src="14_continuum_selfcal/02_baseline_r4_M83_4.png" caption="Baseline, image r4 of M83_4 (colour range −0.2 to 0.45 mJy/beam). M83 is clean in the centre, but sits in a dark negative bowl, and faint oval ripples spread across the whole field." %}

- a **negative bowl** around M83: M83's disk (about 10′ across) has smooth emission larger than the largest clean scale (72″), so it was not fully in the model;
- **oval ripples** across the field, centred on M83: gain errors that change faster than once per scan.

### Try t1: a larger scale (108 px) and a 10-minute amplitude round

Adding the 108 px (3.6′) scale improved every round, and a second amplitude round with a **10-minute** interval removed most ripples. Noise: 0.061 mJy.

{% include fig.html src="14_continuum_selfcal/03_baseline_vs_t1_zoom.png" caption="Zoom on a bright compact source 16′ north-east of M83. Left: baseline. Right: try t1. The rings are almost gone, but a dark collar stays around the source." %}

To find the cause of the collar, look at the **residual** (what clean left behind) and the **model** (what clean found):

{% include fig.html src="14_continuum_selfcal/04_t1_image_residual_model.png" caption="Try t1 at the same source. Top: baseline and t1 images. Bottom left: t1 residual. Bottom right: t1 model." %}

- **Residual:** bright positive flux is still *on* the source: it was not cleaned deep enough. The automatic mask used the *local* noise, which is high around a bright source (because of its own errors), so the clean threshold there was high too.
- **Model:** a ring of broad **negative** components around the source. Clean used them to balance the uncleaned flux. Selfcal then made the data agree with a model with a hole in it.

### Try t2: a proper mask, and no negative components

Two changes: clean inside a mask made from the previous image, down to 1 × the *global* noise (deep on bright sources); and forbid negative components (WSClean `-no-negative`).

{% include fig.html src="14_continuum_selfcal/05_t2_image_residual_model.png" caption="Try t2. Top: t2 and t1 images. Bottom left: t2 residual. Bottom right: t2 model. The bowl around M83 is gone, but the residual is negative across M83, and the model has fuzzy halos around point sources." %}

The bowl disappeared, but this cleaned **too much**: the residual over M83 is negative (the model has *more* flux than the data), M83's spiral arms turned into a smooth blob, and the peak dropped by 4%. Without negative components, broad positive components spread flux into the gaps between the arms, and nothing can correct it.

### Try t3: the mask, with negative components allowed: the winner

Keep the mask, but allow negative components again. Inside the mask they correct over-subtraction; outside the mask nothing is cleaned, so no holes can be dug in empty sky.

{% include fig.html src="14_continuum_selfcal/06_t2_vs_t3_wide.png" caption="Left: try t2. Right: try t3 (colour range −0.1 to 0.7 mJy/beam). In t3 the bright source north-east of M83 is clean (no collar, no rings), M83's spiral arms are back, there is no bowl, and the background is flat." %}

{% include fig.html src="14_continuum_selfcal/07_t3_image_residual_model.png" caption="Try t3. Top: t2 and t3 images. Bottom left: t3 residual. Bottom right: t3 model. The residual is flat almost everywhere; faint positive emission remains only at the edge of M83 and along the tail of the radio galaxy on the left." %}

{: .ours }
> The collar around the compact source looked like a **direction-dependent error** (see below). It was not: it came from the clean model, and a better clean removed it. Always check the residual and the model before blaming the calibration.

### Comparing the tries fairly

| Try | Changes | Image noise (off-source) |
|:----|:--------|-------------------------:|
| baseline | scales to 36 px | 0.073 mJy |
| t1 | + 108 px scale, + 10 min amplitude round | 0.061 mJy |
| t2 | + mask, no negative components | 0.053 mJy |
| **t3** | **+ mask, negative components allowed** | **0.046 mJy** |

{: .problem }
> **The residual noise can fool you.** t2's *residual* noise dropped by 38% after just one phase round, before any amplitude selfcal. That was not better calibration: deeper cleaning simply moves more flux (and some noise) out of the residual into the model. The residual noise measures how much is left uncleaned, not how good the image is.
>
> **Cure:** measure the noise of the final **image**, in empty sky far from all sources. We use a robust estimate, the **MAD** (median absolute deviation × 1.4826), which ignores the few bright pixels. This number does not change just because a run cleans deeper. Also check that peaks do not drop.

## The other six pointings

With the t3 recipe, all six other pointings converged, with no warnings:

| Pointing | r0 (no selfcal) | r1 (phase, per scan) | r5 (final selfcal) | Final image |
|:---------|----------------:|---------------------:|-------------------:|------------:|
| M83_0 | 0.116 | 0.056 | 0.047 | 0.044 |
| M83_1 | 0.076 | 0.050 | 0.047 | 0.044 |
| M83_2 | 0.079 | 0.051 | 0.048 | 0.045 |
| M83_3 | 0.098 | 0.054 | 0.050 | 0.047 |
| M83_4 (t3) | 0.130* | | 0.046 | 0.043 |
| M83_5 | 0.107 | 0.053 | 0.048 | 0.045 |
| M83_6 | 0.141 | 0.061 | 0.054 | 0.052 |

Image noise in mJy/beam. *Residual noise (the off-source measurement did not exist yet).

- For the six outer pointings, **the first phase round did most of the work** (−35 to −57%), and the amplitude rounds only 3–8% more. For M83_4 it was the other way round: M83 itself (about 2 Jy) is in M83_4's main beam, so its amplitude errors dominate there.
- The source peaks did not drop (they rose by 1–4%, because phase errors had smeared them out). Selfcal did not lose flux.
- All pointings end at 1.2–1.4 times the **thermal noise** expected for 10 MHz of bandwidth (about 0.035–0.040 mJy/beam).

{: .ours }
> M83_6 is the noisiest pointing. A short burst of interference near 07:03 UTC (scan 156, M83_6, or 155, M83_5) found in [step 10](../10-apply-and-check/) was never flagged. Selfcal cannot remove interference, so it adds noise. Flagging those few minutes would probably bring M83_6 to the level of the others.

## Final images: a mask for faint extended emission

The t3 residual still shows faint positive emission at the edge of M83 and along a radio galaxy's tail. Each pixel of it is only 1–2 times the noise, so it never entered the 5σ mask and was never cleaned. Together, though, these pixels are clearly real.

For the final image of each pointing (`run_m83_cont_final.slurm`), the mask is made in two ways and combined:

- pixels above **5σ** in the last selfcal image (as before);
- pixels above **4σ** in a **smoothed** copy of the image (a 30″ box, about 3 beams). Smoothing lowers the noise, so faint emission that is spread over several beams rises above the threshold, but single noise peaks do not.

WSClean then cleans inside this mask down to **0.5σ**. This last image does not change the calibration, so it cannot harm it. This brought the noise of each pointing down by another 3–8%.

## Primary beam correction and the mosaic

### The primary beam model: katbeam

Each pointing's image is the sky multiplied by the **primary beam** of the dishes: sources far from the pointing centre look fainter. To get correct brightness, we divide by a model of the beam. Here we use **katbeam**, SARAO's model of the measured MeerKAT beam (`JimBeam('MKAT-AA-L-JIM-2020')`), at 1418 MHz:

| Distance from the pointing centre | Beam |
|:----------------------------------|-----:|
| 0.3° | 0.79 |
| 0.5° | 0.51 (half power) |
| 0.7° | 0.25 |
| 0.9° | 0.08 |

The katbeam beam is slightly oval (at 0.5°: 0.48 in one direction, 0.52 in the other), and it is fixed to the dish. On the sky it turns with the **parallactic angle** (the angle between the dish and the sky, which changes as the source crosses the sky). M83 passes close to the zenith at MeerKAT, so over 11 hours this angle changes by about 180°. We therefore use the beam averaged over all rotations, which is round.

### Combining the pointings

On a common grid (6144 × 2″ = 3.4°), each pixel of the mosaic is

  mosaic = Σ (w<sub>i</sub> B<sub>i</sub> I<sub>i</sub>) / Σ (w<sub>i</sub> B<sub>i</sub><sup>2</sup>),   with w<sub>i</sub> = 1/σ<sub>i</sub><sup>2</sup>

where I<sub>i</sub> is pointing i's image, B<sub>i</sub> its beam and σ<sub>i</sub> its noise. In words: each pointing counts more where its beam is strong and where its noise is low. A pointing is used only where its beam is at least 10% (0.87° from its centre). The noise of the mosaic is 1/√(Σ w<sub>i</sub> B<sub>i</sub><sup>2</sup>), written as a noise map.

{% include fig.html src="14_continuum_selfcal/08_selfcal_mosaic_and_noise.png" caption="Left: the self-calibrated, primary-beam-corrected continuum mosaic (colour range ±0.6 mJy/beam). Right: its noise map (same colour range). The noise is lowest in the middle, where pointings overlap, and rises at the edges." %}

| | Noise |
|:--|--:|
| Step 16 CASA mosaic | 0.22 mJy/beam |
| **Self-calibrated mosaic, centre** | **0.038 mJy/beam** |

- In the centre, the six neighbouring pointings add a little: each has beam ≈ 0.25 at M83, so the weight grows by 6 × 0.25² ≈ 0.375, and the noise drops by 1/√1.375 ≈ 0.85 (0.043 → 0.037 mJy). This is what we measured.
- The faint circles in the noise map are where single pointings stop contributing (their 10% beam edge). They change the noise by only about 1%.
- The negative bowl around M83, the rings near the edges and the faint patches of step 16 are gone.

## What is left, and what could still be done

{: .pending }
> **Direction-dependent calibration: still to check.** Selfcal finds one gain per antenna, for the whole field. Errors that are different in different directions (from the primary beam shape, the pointing of the dishes, or the ionosphere) are largest for bright sources near the edge of a beam, and show as rings or radial "spokes" around them. If bright sources near the mosaic edge show such patterns, **direction-dependent calibration** (with the programs killMS and DDFacet, which solve gains for a few directions separately) would fix them, for those pointings only.

- **Bandwidth:** with 10 MHz, we are close to the thermal limit. A much deeper image needs the full L band (about 856–1712 MHz) of the raw data. That means redoing the calibration for all channels (our calibration tables only cover the 2001 channels we cut out).
- **The HI cube** could also benefit: the selfcal gains could be applied to the line data before continuum subtraction (step 12), which would leave smaller residuals at the positions of continuum sources.

## Other things that can go wrong

{: .problem }
> **Two jobs on the same MS.** A test run and a normal run on the same pointing at the same time would both write `CORRECTED_DATA` and `MODEL_DATA`.
>
> **Cure:** one job per MS at a time. Separate test runs with a tag (`SC_TAG`), which names the images and calibration tables, but they still share the MS.

{: .problem }
> **A Python package that brings its own numpy.** Installing katbeam into a folder for CASA with plain `pip install` would also install numpy there, which can hide CASA's own numpy and break CASA.
>
> **Cure:** `pip install --no-deps --target <folder> katbeam` (katbeam only needs numpy, which CASA already has), then check with CASA's own Python which numpy is loaded.
