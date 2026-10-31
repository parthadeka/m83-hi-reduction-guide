---
title: "16. Continuum image"
parent: Reduction steps
nav_order: 16
permalink: /steps/16-continuum-image/
---

# Step 16. The continuum image
{: .no_toc }

Imaging stage: `continuum` · CASA tasks: `tclean` (mfs, auto-multithresh), `imstat`
{: .step-meta }

1. TOC
{:toc}

---

{: .pending }
> This step was still running when this page was written. The results and plots will be added when it finishes.

## What this step does

This step makes one image of the **radio continuum**: all the emission that is not the HI line. This step is not in the notes; we added it. It is a useful by-product:

- it shows M83's own continuum emission (from star formation and cosmic rays), and the many background radio sources;
- it shows how well the continuum was removed in step 12;
- it is the starting point if we ever want **self-calibration** (see below).

## What we ran

```python
common = dict(vis=vis_hi, imagename='M83_continuum_robust0', spw='0:1250~1420;1580~1800',
              specmode='mfs', weighting='briggs', robust=0.0, deconvolver='hogbom', pbcor=True,
              usemask='auto-multithresh', sidelobethreshold=2.0, noisethreshold=4.25,
              lownoisethreshold=1.5, minbeamfrac=0.3, growiterations=75, parallel=False,
              **MOSAIC)
tclean(niter=0, **common)                                         # dirty image
tclean(niter=100000, threshold=5 * rms0, calcpsf=False, **common)  # clean pass 1
tclean(niter=100000, threshold=3 * rms1, calcpsf=False, **common)  # clean pass 2
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `vis` | the target MS **before** continuum subtraction | Of course: we want the continuum. |
| `spw` | the line-free channels only (`ref_chans`) | The HI channels would add a little line emission to the continuum image. |
| `specmode` | `'mfs'` | **Multi-frequency synthesis**: all channels go into one single image. Each channel has a slightly different uv coverage (baselines are measured in wavelengths), so combining channels also fills the uv plane better. |
| `robust` | `0` | Between natural and uniform. For continuum we want a smaller beam (more detail), and the continuum is bright enough that the extra noise is acceptable. |
| `usemask` | `'auto-multithresh'` | **Automatic masking**: tclean finds the regions with real emission by itself, and only cleans inside them. The mask grows from cycle to cycle. The other parameters are the standard values recommended for this method (the `sidelobethreshold` and `noisethreshold` decide what counts as real emission, relative to the sidelobes and the noise). |
| thresholds | 5 × RMS, then 3 × RMS | We clean in two passes. First down to 5 × the dirty-image RMS (safe: only clearly real emission). Then we measure the RMS of the residual (now lower, because the sidelobes of bright sources are gone) and clean down to 3 × that. |
| `calcpsf` | `False` in the passes | The PSF was already made in the dirty run. Reusing it saves time. |
| `restart` | (default `True`) | Each pass continues from the model of the pass before. |

## What we know so far

- **Dirty image RMS: 0.36 mJy/beam.** This is much higher than the thermal noise we expect for the continuum.
- **Expected thermal noise:** we use 392 line-free channels (about 10 MHz). The noise per channel is about 0.5 mJy, so the noise of 392 channels together is about 0.5 / √392 ≈ **25 µJy**. Robust 0 costs about 1.3–1.5 ×, so we expect **about 35 µJy/beam**.
- **Why is the dirty image 10 times worse?** In a dirty image, every bright source spreads its sidelobes over the whole field. M83 is a very bright continuum source (about 2 Jy), and there are many background sources. Their sidelobes are counted as "noise". Cleaning removes them.
- **Clean pass 1** was going well in a test run: the brightest remaining peak dropped from 40 mJy to 1.8 mJy in five major cycles, and about 4.1 Jy of flux had been cleaned (M83 plus the field sources).

### Running time

This stage is slow: each major cycle takes 40–80 minutes, and the automatic masking adds up to 45 minutes more per cycle. It runs on **one core** (`parallel=False`), because only the cube imaging had been tested in parallel. CASA 6 can also run continuum (`mfs`) imaging in parallel; this would make each major cycle many times faster. **For the next run, switch this stage to parallel.**

## Possible next step: self-calibration

**Self-calibration** ("selfcal") uses the target field itself as its own calibrator. We make a good continuum model of the field (this step), then solve for the gains again by comparing the data with that model, at short time intervals. Because the target field is in the right direction at all times, selfcal can correct errors that the gain calibrator could not (for example, from the different atmosphere in the direction of M83, or from changes between calibrator visits).

Selfcal is not in the notes, and it may not be needed. The decision rule:

- If the **final continuum RMS** is close to the expected ~35–40 µJy, and there are no strong artefacts (rings, stripes, "spokes") around M83 and bright sources, selfcal is not needed for HI work.
- If the RMS stays well above ~100 µJy, or the cube's line-free channels show artefacts around bright continuum sources, selfcal is worth doing. It is done on the target MS, before continuum subtraction, and then steps 12–15 are repeated.

## What can go wrong

{: .problem }
> **Two jobs writing the same image.** In our run, a separate job making this continuum image crashed with "table.f0 does not exist and cannot be created", because the main imaging job reached its own continuum stage and deleted the half-finished images to start fresh.
>
> **Cure:** never run the same imaging stage in two jobs at the same time.

{: .problem }
> **Artefacts around bright sources** (rings, radial "spokes", stripes). These come from calibration errors (often direction-dependent: the primary beam, or the ionosphere differs across the field).
>
> **Cure:** self-calibration; for the strongest cases, direction-dependent calibration (beyond this guide).

{: .problem }
> **Mask too small or too large.** auto-multithresh can miss faint extended emission, or include sidelobes.
>
> **Cure:** check the `.mask` image after cleaning; adjust `noisethreshold` and `sidelobethreshold`, or draw a mask by hand.

{: .problem }
> **Wide-band effects.** Over large bandwidths, sources change brightness with frequency (spectral index). With only 10 MHz this does not matter; for a full-band (800 MHz) MeerKAT continuum image you must use `deconvolver='mtmfs'` with `nterms=2`.
