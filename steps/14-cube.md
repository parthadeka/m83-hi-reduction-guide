---
title: "14. The HI data cube"
parent: Reduction steps
nav_order: 14
permalink: /steps/14-cube/
---

# Step 14. The HI data cube
{: .no_toc }

Imaging stages: `cube`, `cube_no_contsub` · CASA task: `tclean` (parallel, with `mpicasa`)
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

We make the **HI data cube**: a 3D image with two sky axes (right ascension and declination) and one **spectral axis** (frequency, or velocity). Each plane of the cube is one channel: an image of the gas moving at one particular velocity.

One element of a cube is called a **voxel** ("volume pixel"): one sky position in one channel.

## What we ran

```python
tclean(vis=vis_hi_sub, imagename='M83_HI_cube_contsub_robust1',
       specmode='cube', start=1420, nchan=160, width=1, outframe='LSRK',
       restfreq='1420405752Hz', weighting='briggs', robust=1.0, deconvolver='hogbom',
       niter=100000, threshold='1.091mJy', pbcor=True, restoringbeam='common',
       parallel=True, **MOSAIC)
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `specmode` | `'cube'` | Make one image per channel (instead of one image for all channels together). |
| `start`, `nchan`, `width` | 1420, 160, 1 | Image MS channels 1420 to 1579, **each native channel separately** (`width=1`, no averaging). This is `tar_chans` from the notes: M83's HI plus empty margins on both sides. |
| `outframe` | `'LSRK'` | The **reference frame** of the spectral axis. The telescope measures frequency in its own frame (TOPO, "topocentric"), which moves with the Earth's rotation and orbit. LSRK ("kinematic local standard of rest") removes the Earth's motion, so velocities can be compared between observations and with the literature. tclean shifts each visibility to the right LSRK channel. |
| `restfreq` | 1420.405752 MHz | The rest frequency of the HI line. Needed to convert frequency into velocity. |
| `weighting`, `robust` | Briggs, 1 | Close to natural: good sensitivity to extended gas, with smaller sidelobes than pure natural (see [step 13](../13-imaging-setup/)). As in the notes. |
| `deconvolver` | `'hogbom'` | The classic CLEAN algorithm: point-like clean components. (`'multiscale'` uses components of several sizes and is often better for very extended emission.) |
| `niter` | 100000 | The maximum number of clean iterations. In practice the threshold stops the cleaning first. |
| `threshold` | 1.09 mJy | 2 × the measured noise (step 13). |
| `pbcor` | `True` | Also write a **primary-beam-corrected** image (`.image.pbcor`): the image divided by the primary beam response. This gives the true brightness everywhere, but the noise rises towards the edges. |
| `restoringbeam` | `'common'` | The beam size changes slightly with frequency. `'common'` restores all channels with the same beam (the largest one), so all channels have the same resolution. This is needed for moment maps (step 15). |
| `parallel` | `True` | Split the channels over several CPU cores (16, using `mpicasa`). Each core images different channels. |

### The velocity axis of our cube

| Cube channel | Velocity (radio, LSRK) |
|:------------:|:-------------------------:|
| 0 | about 1043 km/s |
| ~67 | ~670 km/s (high-velocity edge of M83) |
| **~96** | **~513 km/s (M83's systemic velocity)** |
| ~124 | ~360 km/s (low-velocity edge of M83) |
| 159 | about 170 km/s |

Each channel is **5.5 km/s** wide. Frequency increases along the cube, so velocity *decreases*. M83's HI is in cube channels **~65 to ~125**. Channels 0–40 and 140–159 should contain only noise.

The **systemic velocity** is the velocity of the galaxy as a whole. Because the galaxy rotates, one side of the disk moves towards us (lower velocity) and the other away from us (higher velocity), relative to the systemic velocity.

### Two cubes

We make two cubes with the same settings:

1. **`M83_HI_cube_contsub_robust1`** from the continuum-subtracted data. This is the **science cube**.
2. **`M83_HI_cube_no_contsub_robust1`** from the data *before* continuum subtraction. This cube still contains M83's continuum and all background sources. It is useful to check the continuum subtraction, and to look for **HI absorption** against continuum sources (absorption is easier to measure when the continuum it absorbs is still there).

## How long it took

The cube is 4096 × 4096 pixels × 160 channels. tclean reported that the full cube would need about **5 TB of memory**, so it split the work into **45 sub-cubes**, spread over 15 worker processes. Making the PSF took about **28 minutes**, and each major cycle (one pass through all the data) a similar time. Both cubes together, plus the moment maps, took about **10 hours** on 16 cores.

{: .plain }
> A **major cycle** is one pass through all the visibilities, to make a new residual image. Between major cycles, **minor cycles** find and remove clean components in the image. tclean alternates between the two until the threshold is reached.

## Plots

The cube itself is best viewed with an interactive viewer, such as **CARTA** (Cube Analysis and Rendering Tool for Astronomy), which ilifu provides. Open `images/02_cube_contsub/M83_HI_cube_contsub_robust1.image` and step through channels 65–125. Set the spectral axis to "radio velocity (km/s)".

The moment maps and the global profile made from this cube are shown in [step 15](../15-moments/).

## What can go wrong

{: .problem }
> **Wrong velocity range.** If the cube does not cover all of the galaxy's velocities, the edges of the line are cut off.
>
> **Cure:** check the global profile (step 15): it must fall to zero on both sides, inside the cube.

{: .problem }
> **Wrong reference frame.** Mixing TOPO and LSRK velocities gives shifts of up to ~30 km/s (5–6 channels).
>
> **Cure:** always image in LSRK (or BARY), and say which frame you use.

{: .problem }
> **Cleaning not deep enough, or too deep.** Too shallow: sidelobes remain around bright emission; the flux in the image is wrong. Too deep: noise is cleaned, and false structure appears.
>
> **Cure:** a threshold of 2–3 × RMS; a **clean mask** (only clean where there is real emission); or automatic masking (`usemask='auto-multithresh'`).

{: .problem }
> **Negative bowls around extended emission.** If large-scale emission is not cleaned well, the image has negative areas around it.
>
> **Cure:** clean deeper inside a mask, or use multiscale clean (`deconvolver='multiscale'`, with `scales=[0, 5, 15, 45]` pixels, for example).

{: .problem }
> **Out of memory or very slow.** Large cubes need a lot of memory and time.
>
> **Cure:** parallel imaging with `mpicasa` (as we did); fewer channels; smaller images; or chunking the cube by channel into separate jobs and combining afterwards with `imageconcat`.

{: .problem }
> **Beam changes from channel to channel.** Without `restoringbeam='common'`, each channel has a slightly different beam, and moment maps or flux sums become inaccurate.
