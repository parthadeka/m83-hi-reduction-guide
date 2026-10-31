---
title: "13. Imaging: setup and noise test"
parent: Reduction steps
nav_order: 13
permalink: /steps/13-imaging-setup/
---

# Step 13. Imaging: setup and noise test
{: .no_toc }

Imaging stages: `prep`, `noise_image` · CASA tasks: `vpmanager`, `tclean`, `imstat`
{: .step-meta }

1. TOC
{:toc}

---

## From visibilities to an image

An interferometer measures visibilities in the **uv plane** (see [step 2](../02-first-look/)). The sky image and the visibilities are linked by a mathematical operation called the **Fourier transform**. To make an image, CASA:

1. puts every visibility on a regular grid in the uv plane (**gridding**);
2. Fourier transforms the grid into an image.

The result is the **dirty image**. It is "dirty" because our uv coverage has gaps. Every source in a dirty image is surrounded by the pattern of the **dirty beam** (also called the **point spread function**, **PSF**, or **synthesised beam**): the image that a single point source would give. The dirty beam has a central peak plus **sidelobes** (rings and ripples).

**Cleaning** (deconvolution) removes the sidelobes. The CLEAN algorithm repeatedly finds the brightest point in the image, records a small piece of flux there (a **clean component**) in a **model**, and removes that point's dirty-beam pattern from the image. It stops at a chosen **threshold**. At the end, the model is smoothed with a clean Gaussian beam (the **restoring beam**) and the leftover image (the **residual**) is added back.

CASA's imaging task is **`tclean`**.

## Part 1: the primary beam model (`prep`)

Each antenna is most sensitive in the direction it points, and less sensitive further away. This sensitivity pattern is the **primary beam** (PB). For a mosaic, CASA must know its shape exactly: it uses the PB to combine the seven pointings correctly, and to correct the brightness at the edges.

CASA has no built-in MeerKAT beam, so we create one, using the formula from the notes (a "cosine-taper" model, from Mauch et al. 2020):

```python
def beam(offset, freq):
    fwhm = sqrt(89.5 * 86.2) / 60. * (1e3 / freq)    # degrees; freq in MHz
    x = 1.189 * offset / fwhm
    return (cos(pi * x) / (1 - 4 * x**2))**2

vp.setpbnumeric(telescope='OTHER', othertelescope='MeerKAT', vect=beam(...), maxrad='3.0deg',
                reffreq='1.4GHz', isthispb='PB', ...)
vp.saveastable('MEERKAT_PB.tab')
```

At 1.4 GHz this gives a FWHM of about **63 arcminutes** (1.05 degrees). CASA scales the beam with frequency automatically. The table is passed to every `tclean` call as `vptable`.

{: .problem }
> **Wrong primary beam.** A beam that is too wide or too narrow gives wrong fluxes away from the pointing centres, and seams between pointings in the mosaic.
>
> **Cure:** use a measured model for your telescope and band.

## Part 2: common imaging settings

| Parameter | Value | Why |
|:----------|:------|:----|
| `gridder` | `'mosaic'` | Combines all seven pointings into one image while gridding, using the primary beam of each pointing. |
| `field` | all seven M83 pointings | |
| `phasecenter` | `'J2000 13h37m00.95 -29d51m55.5'` | The centre of the output image: the centre pointing, M83_4, on M83's nucleus. |
| `imsize` | `4096` pixels | 4096 × 3″ = 3.4 degrees. The mosaic is about 2–2.5 degrees across, plus the primary beam edges. A power of 2 (or a number with only small prime factors) makes the Fourier transform fast. |
| `cell` | `'3arcsec'` | The pixel size. The rule is to have at least 3–5 pixels across the beam. Our beam is about 25–30″, so 3″ gives 8–10 pixels per beam. |
| `pblimit` | `0.1` | Do not image where the combined primary beam response is below 10%. Outside that, the noise becomes very large. |
| `stokes` | `'I'` | Total intensity (the average of XX and YY). |
| `savemodel` | `'none'` | Do not write the clean model back into the MS. This keeps the `MODEL_DATA` column (which holds the continuum fit from step 12) untouched. |

### Weighting

When CASA grids the visibilities, it can give each one a different **weight**. This choice changes the beam and the noise:

- **Natural weighting** gives every visibility the same weight. Because the core of MeerKAT is dense, the short baselines dominate. Result: **lowest noise**, but a **larger beam** (less detail) and broad sidelobes.
- **Uniform weighting** gives every *uv cell* the same weight, so that short and long baselines count equally. Result: **smaller beam** (more detail), but **higher noise**.
- **Briggs (robust) weighting** sits between the two. The `robust` parameter goes from −2 (close to uniform) to +2 (close to natural).

For HI, we usually want sensitivity to faint, extended gas, so we stay close to natural. The notes used **Briggs with robust = 1**, and so do our cubes.

## Part 3: the noise test (`noise_image`)

### What it does

Before we spend many hours making the full cube, we make a **dirty image of a single, line-free channel** (channel 1440, at about 913 km/s, far from M83's HI). Because it is line-free and continuum-subtracted, it should contain only noise. We measure that noise and compare it with the prediction from [step 11](../11-split-and-noise/).

We make it twice: with **natural** weighting (to compare with the theory, as the notes did) and with **robust 1** (the weighting of our cube, to set the clean threshold).

```python
tclean(vis=vis_hi_sub, imagename=..., specmode='cube', start=1440, nchan=1,
       weighting='natural', niter=0, **MOSAIC)
imstat(imagename=... + '.image', box='1000,1000,3000,3000')
```

`niter=0` means "no cleaning": just the dirty image. The `box` measures the statistics in the central 2000 × 2000 pixels.

### Results

{% include fig.html src="13_imaging/noise_chan1440_natural.png" caption="Dirty image of line-free channel 1440, natural weighting. RMS noise 0.527 mJy/beam." %}

{% include fig.html src="13_imaging/noise_chan1440_robust1.png" caption="The same channel with Briggs robust = 1. RMS noise 0.545 mJy/beam." %}

| | Measured RMS | Beam |
|:--|:--:|:--:|
| Theory, one pointing (step 11) | 0.50 – 0.55 mJy | |
| The notes (2019, natural) | 0.547 mJy | |
| **Our natural image** | **0.527 mJy/beam** | 24.8″ × 22.3″ |
| **Our robust 1 image** | **0.545 mJy/beam** | 29.5″ × 26.0″ |

**RMS** ("root mean square") is the standard way to measure noise: roughly, the typical size of the random ups and downs.

**What we see:**

- **The measured noise matches the theory.** This is the most important result of the whole reduction so far. It means the calibration is good, the flagging removed the bad data, and the continuum subtraction left nothing behind in this channel.
- **The images look like pure noise**: random red and blue blobs of about the beam size, with no sources, stripes or rings.
- **The hexagon-with-rounded-edges shape** is the mosaic: seven circles (one per pointing) cut at the 10% primary beam level.
- In the robust-1 image, the noise looks **finer-grained near the centre** than near the edges. In the centre, all pointings overlap, and the combination of their weights changes the effective uv weighting a little there.

{: .note }
> Normally robust weighting gives a *smaller* beam than natural weighting. Here the robust-1 beam came out slightly *larger*. This is unusual. A likely reason is that the fitted Gaussian does not describe MeerKAT's non-Gaussian natural beam well (MeerKAT's dense core gives the natural beam a narrow peak on top of a broad "plateau"). It is worth looking at the `.psf` images to understand it. It does not change the conclusions here.

### The clean threshold

From this test, the script sets the clean threshold for the cubes:

> threshold = 2 × RMS (robust 1) = 2 × 0.545 mJy = **1.09 mJy**

Cleaning deeper than about 2–3 times the noise starts to "clean" noise peaks, which adds false structure. Stopping much higher leaves sidelobes of bright emission in the cube. The notes used 2 × RMS as well.

## What can go wrong

{: .problem }
> **Noise much higher than expected.** See the list in [step 11](../11-split-and-noise/). Also check that the channel really is line-free and continuum-free.

{: .problem }
> **Visible sources or stripes in a "line-free" channel.** The continuum subtraction did not work well (see [step 12](../12-continuum-subtraction/)), or the channel contains a line.

{: .problem }
> **Image too small.** Sources outside the image still put their sidelobes into it, and they cannot be cleaned.
>
> **Cure:** make the image cover the whole primary beam (or all pointings' beams), as we did.

{: .problem }
> **Pixels too large.** Fewer than ~3 pixels across the beam makes cleaning inaccurate.

{: .problem }
> **Out of memory.** A 4096 × 4096 mosaic needs a lot of memory. CASA reports "Required memory" in the log and splits the work into pieces when needed.
