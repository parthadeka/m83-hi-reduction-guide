---
title: "15. Moment maps"
parent: Reduction steps
nav_order: 15
permalink: /steps/15-moments/
---

# Step 15. Moment maps
{: .no_toc }

Imaging stage: `moments` · CASA tasks: `immoments`, `imstat`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

A cube has 160 channels. It is hard to look at all of them at once. **Moment maps** summarise the cube in a few 2D images. For each sky pixel, we look at its spectrum (the column of voxels along the velocity axis) and calculate:

| Moment | Name | What it tells us | Unit |
|:------:|:-----|:-----------------|:-----|
| **0** | Integrated intensity | **How much gas** there is: the spectrum summed over velocity | Jy/beam × km/s |
| **1** | Intensity-weighted velocity | **Where the gas moves**: the average velocity of the gas | km/s |
| **2** | Velocity dispersion | **How spread out** the velocities are along the line of sight | km/s |

The **moment 0** map is proportional to the **column density** of HI: the number of hydrogen atoms in a column through the galaxy. The **moment 1** map is the **velocity field**: it shows how the galaxy rotates.

## Noise and clipping

When we add up all channels, we also add up the noise of all channels. In the channels without HI, we only add noise. A common simple fix is **clipping**: only include voxels brighter than some limit.

```python
rms = imstat(imagename=cube + '.image', chans='0~9;150~159')['rms'][0]
immoments(imagename=cube + '.image', moments=[0], outfile=mom_base + '.mom0')
immoments(imagename=cube + '.image', moments=[0, 1, 2],
          excludepix=[-1000, 3 * rms], outfile=mom_base + '_cut3sigma.mom')
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `chans` (for the RMS) | 0–9 and 150–159 | The first and last 10 channels of the cube are far from M83's HI, so they are pure noise. |
| `excludepix` | [−1000, 3 × RMS] | **Exclude** voxels with values between −1000 and 3 × RMS. In practice: keep only voxels above **3 σ** (σ is the RMS noise). |

We make:

1. a moment 0 map **without** any clipping (it keeps all flux but is very noisy);
2. moment 0, 1 and 2 maps **with** the 3σ clip;
3. a **primary-beam-corrected** moment 0, only where the cube is above 3σ.

We also sum the cube over the sky to make the **global profile** (step below).

## The maps

### Moment 0: how much gas

{% include fig.html src="13_imaging/mom0_pbcor_3sigma.png" caption="M83 HI moment 0, primary-beam corrected, voxels above 3σ. Axes are pixels of 3 arcseconds." %}

**What we see:**

- **The bright inner disk** with clear **spiral arms**, following M83's optical spiral structure.
- **Long, faint outer arms and a ring** of HI far outside the bright disk, reaching about 30–40 arcminutes from the centre. The HI disk is several times larger than the optical galaxy. This is why the observation used a mosaic.
- **A long northern structure** (the arc at the top of the disk) and **gas extending to the south-west**.
- **Three small, separate HI sources**: one to the left (east) at about pixel (500, 1550), and two to the north, at about (830, 2780) and (1830, 2370). These are very probably **dwarf galaxies** near M83, or background galaxies. A source finder (step 17) will catalogue them properly.
- **Speckles everywhere** in the background. These are **noise peaks** that happened to be above 3σ. In a cube of 2.7 billion voxels, millions of noise voxels are above 3σ just by chance. This is the main weakness of simple clipping.

### Moment 1: how the gas moves

{% include fig.html src="13_imaging/mom1_3sigma.png" caption="M83 HI moment 1 (velocity field), voxels above 3σ." %}

**What we see:** a smooth change of velocity across the disk: about **400 km/s (blue)** in the north and **650 km/s (red-white)** in the south. This is **rotation**: one side of the disk moves towards us, the other away, relative to the systemic velocity of ~513 km/s. The lines of equal velocity bend in an "S" shape near the centre, which is typical of a rotating disk with a **warp** or non-circular motions (for example along the bar of M83). Away from the galaxy, the colours are random: that is noise.

### Moment 2: how spread out the velocities are

{% include fig.html src="13_imaging/mom2_3sigma.png" caption="M83 HI moment 2 (velocity dispersion), voxels above 3σ." %}

**What we see:** mostly **10–40 km/s** across the disk (dark blue/purple). This is typical for HI in spiral galaxies (thermal motions plus turbulence, broadened a little by the beam and by rotation within the beam). Higher values appear near the centre, where the rotation changes fast within one beam. In the noise, moment 2 has no meaning.

### Global HI profile

{% include fig.html src="13_imaging/global_profile_3sigma.png" caption="Global HI profile: the flux of all voxels above 3σ, summed over the sky, against velocity (radio, LSRK)." %}

**How to read it.** For each channel, add up the flux of the whole galaxy. The result is the total HI spectrum of M83, as a single-dish telescope would see it.

**What we see:**

- HI from about **355 to 660 km/s**: a total width of about 300 km/s.
- A **double-horned shape**, the classic signature of a rotating disk: much of the gas moves near the maximum rotation speed, towards or away from us, so the spectrum has peaks near both edges.
- The two horns are **not equal**: the high-velocity side is brighter. The two sides of the galaxy contain different amounts of gas (M83 is lopsided, with more gas in some outer arms).
- **The baseline is at about 650 mJy, not at zero.** This is the **clipping bias**. In every channel, including those without HI, the 3σ clip keeps the positive noise peaks and throws away all negative values. Adding millions of positive noise peaks over the whole 3° image gives a positive "floor". The true profile is zero outside the line.

{: .ours }
> The clipping bias also affects the moment 0 map and any total flux measured from it. **Do not use these quick-look clipped maps for flux measurements.** Use the SoFiA masks (step 17) instead.

## Why clipping is not good enough

| Problem | Effect |
|:--------|:-------|
| Faint, extended HI is lost | Each voxel of the faint outer disk is below 3σ, even though together they are clearly real |
| Noise peaks are kept | Speckles in mom0; random values in mom1 and mom2 |
| Only positive noise is kept | Fluxes are biased high (the 650 mJy floor) |

A better approach is a **3D mask**: decide which voxels belong to real sources by looking at their neighbours in position *and* velocity, and at several smoothing scales. Inside the mask, keep *all* voxels (positive and negative); outside, keep none. This is what **SoFiA** does (step 17).

## What can go wrong

{: .problem }
> **Moment 1 and 2 dominated by noise.** Because they divide by moment 0, they are very sensitive to noise where there is little signal.
>
> **Cure:** use a proper mask (SoFiA), or a higher clip, or smooth the cube first.

{: .problem }
> **Wrong RMS.** If the channels used for the RMS contain emission, the RMS is too high and the clip removes real gas.
>
> **Cure:** check that the noise channels are really empty (here: cube channels 0–9 and 150–159).

{: .problem }
> **Using the primary-beam-corrected cube to find sources.** Its noise is much higher at the edges, so a fixed clip level keeps too many noise voxels there.
>
> **Cure:** find sources (or make the clip mask) on the **non-corrected** cube, which has flat noise, then apply that mask to the corrected cube for fluxes. Our pb-corrected moment 0 does exactly this.
