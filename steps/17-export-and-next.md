---
title: "17. Export and next steps (SoFiA)"
parent: Reduction steps
nav_order: 17
permalink: /steps/17-export-and-next/
---

# Step 17. Export to FITS, and next steps with SoFiA
{: .no_toc }

Imaging stage: `export` · CASA task: `exportfits` · Next: SoFiA-2
{: .step-meta }

1. TOC
{:toc}

---

## Exporting to FITS

CASA stores images in its own format (a folder, like an MS). Most other astronomy software reads **FITS** (Flexible Image Transport System), the standard file format for astronomical images. The last imaging stage converts the important images to FITS:

```python
exportfits(imagename=img, fitsimage=img + '.fits', dropstokes=True, overwrite=True)
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `dropstokes` | `True` | Our images have a fourth axis for Stokes (polarisation), with only one value (I). Many programs (including SoFiA) prefer a plain 3D cube (RA, Dec, frequency). |
| `overwrite` | `True` | Replace older FITS files when the stage is run again. |

The files are written to `images/fits/`: both cubes (`.image`, `.image.pbcor`, `.pb`), the moment maps, and the continuum image.

## Next: finding the HI with SoFiA

[Step 15](../15-moments/) showed that simple 3σ clipping is not good enough: it loses faint gas, keeps noise peaks, and biases the fluxes. The standard solution is a **source finder**. The one used in the notes, and widely used for MeerKAT HI data, is **SoFiA** (the Source Finding Application), version 2.

### What SoFiA does

1. **Smooth and clip** (S+C): it smooths the cube with several sizes of kernel, in position (for example 0, 3 and 6 pixels) and in velocity (0, 3, 7 and 15 channels). After each smoothing, it keeps voxels above a threshold (default 5 σ). A faint, extended cloud that is invisible in the original cube becomes clear when smoothed.
2. **Linking**: neighbouring detected voxels are joined into **sources**.
3. **Reliability**: noise produces as many negative "sources" as positive ones. SoFiA compares the two populations and removes positive detections that look like noise.
4. **Mask and parameters**: the final 3D **mask** says which voxels belong to which source. Inside the mask, *all* voxels are used (positive and negative), so fluxes are not biased. SoFiA then measures each source and writes a **catalogue**.

### What SoFiA gives us

- a **catalogue** of all HI sources in the cube: M83, and probably its dwarf companions (the three compact sources seen in the moment 0 map);
- for each source: position, **total flux** (from which we get the **HI mass**), **systemic velocity**, line widths **W20** and **W50** (the width of the profile at 20% and 50% of the peak), and orientation;
- clean **moment 0, 1 and 2 maps**, without noise speckles;
- **cubelets**: small cubes around each source;
- the **mask cube**, which can be used on other cubes.

### How we set it up

Two parameter files, run one after the other (`run_m83_sofia.slurm`):

1. **`sofia_M83_contsub.par`**: run SoFiA on the **non-primary-beam-corrected** cube (`M83_HI_cube_contsub_robust1.image.fits`). This cube has flat noise across the mosaic, which is what a source finder needs. All detection settings are SoFiA's defaults, as in the notes. Moments and cubelets are written, as in the notes.
2. **`sofia_M83_contsub_pbcor.par`**: apply the mask from run 1 to the **primary-beam-corrected** cube, without searching again. This gives correct fluxes everywhere, without the false detections that the higher edge noise would cause.

The notes used SoFiA **2.1.0** (2019). The ilifu cluster provides SoFiA-2 as a container: `/idia/software/containers/sofia-latest.sif`, which is version **2.6.7**. A few default values have changed since 2.1. If exact agreement with the notes is needed, the 2.1.0 defaults can be written into the parameter file.

### Possible problems with SoFiA

{: .problem }
> **Noise is not uniform.** If the noise varies across the cube (for example in a primary-beam-corrected cube, or near bright continuum residuals), a single threshold gives false detections in noisy areas and misses sources in quiet ones.
>
> **Cure:** run on the flat-noise cube; or switch on SoFiA's noise scaling (`scaleNoise.enable = true`).

{: .problem }
> **Continuum residuals or artefacts are detected as sources.**
>
> **Cure:** improve continuum subtraction or calibration (selfcal); use the reliability filter; check every detection by eye.

{: .problem }
> **Not enough memory.** SoFiA keeps several copies of the cube in memory. Our cube is about 11 GB.
>
> **Cure:** request enough memory (we ask for 128 GB), or process a sub-region (`input.region`).

## Where things are on the cluster

All outputs are under `/idia/projects/hixobz/M83/`:

| Folder | Contents |
|:-------|:---------|
| `data/` | The MSs: working MS, target MS, continuum-subtracted target MS |
| `caltables/` | All calibration tables, and the MeerKAT primary beam table |
| `plots/` | All plots shown in this guide |
| `reports/` | Text and JSON summaries: listobs, flags, delays, fluxes, noise |
| `images/` | Noise test, cubes, moments, continuum, FITS files |
| `sofia/` | SoFiA outputs |
| `selfcal/` | Continuum self-calibration ([step 18](../18-continuum-selfcal/)): per-pointing MSs, images, calibration tables, reports, and the final mosaic |
| `logs/` | CASA and slurm logs |
| `script/` | The scripts and parameter files |

## Summary of the whole reduction

| Check | Result |
|:------|:-------|
| Flagged data | Only a few percent (autocorrelations, m003, shadowing, Milky Way HI on the calibrators) |
| Delays | Below 0.5 ns, stable |
| Bandpass | Smooth, ±4% slope; one antenna (m001) with a harmless ripple |
| Gains | Smooth over the night; quiet ionosphere |
| PKS1313-333 flux | 0.724 ± 0.004 Jy (notes: 0.723 Jy) |
| Flux calibrator after calibration | 14.85–14.89 Jy across 1411–1425 MHz, matching the model (setjy gave 14.95 Jy; the model falls slowly with frequency) |
| Remaining problem | One short spike in one target scan near 07:03 UTC; to be flagged |
| Line-free channel noise | 0.527 mJy/beam (natural), as predicted (0.50–0.55 mJy) |
| HI detected | 355–660 km/s; extended disk, outer arms, and three compact companions |
| Continuum image (step 16) | 0.22–0.24 mJy/beam (thermal limit ~0.035); limited by artefacts: a negative bowl around M83 and rings around bright off-centre sources |
| Continuum after self-calibration (step 18) | 0.038 mJy/beam in the centre of the mosaic, close to the thermal limit; no bowl, no rings |
| HI absorption at the nucleus (step 19) | About 9.5 mJy absorbed against a compact source (< 5″); τ ≥ 0.37, N<sub>HI</sub> ≥ 3.8 × 10<sup>21</sup> cm<sup>−2</sup> for T<sub>s</sub> = 100 K; centred at the systemic velocity |
