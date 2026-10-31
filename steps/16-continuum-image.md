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
tclean(niter=100000, threshold=3 * rms1, calcpsf=False, **common)  # clean pass 2 (not reached, see below)
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

## What happened in our run

### Clean pass 1 stalled

| Major cycle | Components cleaned | Model flux | Peak residual after the major cycle |
|:--|--:|--:|--:|
| 1 | 259 | 1.09 Jy | 140 mJy |
| 2 | 1438 | 2.44 Jy | 40 mJy |
| 3 | 6675 | 3.91 Jy | 12.9 mJy |
| 4 | 1521 | 4.12 Jy | 4.4 mJy |
| 5 | 81 | 4.11 Jy | 2.7 mJy |
| 6 | 9 | 4.11 Jy | 2.6 mJy |

Each minor cycle reached the threshold (1.8 mJy). But after every major cycle, the peak residual was back at about 2.5 mJy, so tclean started another major cycle (about 2.5 hours each), cleaned only a few components, and repeated. The model flux stopped growing.

{: .plain }
> The remaining 2.5 mJy peaks are not real sky emission that cleaning could remove. They are **artefacts** from calibration errors: 2.5 mJy out of 4 Jy is a dynamic range of about 1:1600, a typical limit for an image without self-calibration. Cleaning cannot remove calibration errors, so the clean would never reach its threshold. Pass 2, with an even lower threshold, would have run for days.

We stopped the job, and made the final image from the model and residual already on disk, without any more cleaning (a separate stage, `continuum_restore`: `tclean` with `niter=0, calcres=False, calcpsf=False, restart=True`).

{: .problem }
> **A clean that never ends.** If the threshold is below the level of the artefacts, tclean keeps running major cycles that remove almost nothing.
>
> **Cure:** watch the log: if the number of components per cycle drops to a few, and the peak residual stops falling, stop. Set `nmajor` (a maximum number of major cycles), use `nsigma` instead of a fixed threshold, and fix the calibration (self-calibration) before cleaning deeper.

### The image

{% include fig.html src="13_imaging/continuum_robust0.png" caption="Continuum image of the line-free channels (robust 0), after clean pass 1. Colour scale −0.5 to 1.2 mJy/beam, so M83 (about 2 Jy in total) is strongly saturated. Axes are pixels of 3 arcseconds (the image is cropped to the mosaic)." %}

**Noise:** about **0.22–0.24 mJy/beam** (dirty image: 0.36 mJy/beam). The thermal noise we expect for 10 MHz of bandwidth is about 0.035 mJy/beam, so the noise is still **6–7 times** higher than the thermal limit. The image is limited by artefacts, not by the receivers.

**What we see:**

- **M83** in the centre, bright and saturated, with its spiral disk visible.
- **A dark, negative ring around M83** (radius about 7–10 arcminutes). This is a **negative bowl**: M83 has faint, extended continuum emission that was not fully cleaned (the automatic mask covered only the bright parts, and `hogbom` cleans with point-like components, which is poor for extended emission). The uncleaned extended emission leaves negative sidelobes around the galaxy.
- **Rings and ripples around bright sources far from the centre** (for example on the left and right edges, and at the bottom). These sources are in the outer parts of the primary beams, where small errors in the beam model, in the pointing, or in the ionosphere are largest. These are **direction-dependent errors**: the gain calibration is correct in one direction (on average) but not for every direction in the field.
- **Large, faint patches** brighter and darker than average across the field: sidelobes of all the sources below 1.8 mJy that were never cleaned.
- **Hundreds of background radio sources**, most of them distant galaxies (active galactic nuclei). Some are **extended radio galaxies**: a curved source with a tail to the left of M83, a hook-shaped source in the north, and a double source on the left. Their shapes come from jets of plasma bent by the gas they move through.

{: .ours }
> For the HI work, this image is good enough: it shows where the continuum is, and how bright it is. As a science image of M83's continuum it is not competitive yet: the ASKAP EMU survey reaches about 0.025–0.03 mJy/beam in this part of the sky.

### How to improve it

| Problem | Cure |
|:--|:--|
| Negative bowl around M83 | `deconvolver='multiscale'` (for example `scales=[0, 5, 15, 45]` pixels), which cleans with components of several sizes, and a larger mask around M83 |
| Rings around bright sources | Self-calibration (below); for sources far out in the beam, direction-dependent calibration |
| Many faint sources never cleaned | Clean deeper, after self-calibration, with `nsigma` and `nmajor` limits |
| Very slow | `parallel=True` (with mpicasa), or WSClean, which is much faster for continuum |
| Only 10 MHz of bandwidth | Use the full L band (about 856–1712 MHz) from the raw data, with `deconvolver='mtmfs'` |

### Running time

Each major cycle took about 1 h 45 min, and the automatic masking up to 45 min more. It ran on **one core** (`parallel=False`), because only the cube imaging had been tested in parallel. CASA 6 can also run continuum (`mfs`) imaging in parallel; this would make each major cycle many times faster. **For the next run, switch this stage to parallel.**

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
