---
title: "20. SoFiA results"
parent: Reduction steps
nav_order: 20
permalink: /steps/20-sofia-results/
---

# Step 20. SoFiA results: M83 and its neighbours
{: .no_toc }

Software: SoFiA 2.6.7 (`sofia_M83_contsub.par`, `sofia_M83_contsub_pbcor.par`) · Plots: `sofia_plots.py`
{: .step-meta }

1. TOC
{:toc}

---

## What we ran

As planned in [step 17](../17-export-and-next/), SoFiA ran twice:

1. **Finding** on the continuum-subtracted cube **without** primary beam correction (flat noise), with SoFiA's default settings, as in the notes.
2. **Measuring**: the mask from run 1 applied to the **primary-beam-corrected** cube, without searching again. All maps and numbers on this page come from this run.

The plots are made by `sofia_plots.py` (Python with numpy, matplotlib and astropy):

```bash
python3 sofia_plots.py                      # default paths of this reduction
python3 sofia_plots.py --distance 4.66      # another distance for the HI masses
```

### From SoFiA's units to physical units

SoFiA works in the units of the cube: Jy/beam, and **frequency** in Hz (our cube's third axis). The script converts:

| Quantity | Conversion |
|:---------|:-----------|
| Velocity | v = c (1 − ν / ν<sub>0</sub>), with ν<sub>0</sub> = 1420.405752 MHz (the "radio" velocity convention, in the LSRK frame) |
| Widths, dispersion | 1 Hz = c / ν<sub>0</sub> = 2.11 × 10<sup>−4</sup> km/s; one channel = 26.12 kHz = 5.51 km/s |
| Total flux S<sub>int</sub> | `f_sum` is a **sum of Jy/beam pixel values**. Divide by the beam area in pixels (96.6 for our 29.5″ × 26.0″ beam and 3″ pixels) and multiply by the channel width: Jy km/s |
| Column density | N<sub>HI</sub> = 1.104 × 10<sup>21</sup> × S [mJy/beam km/s] / (b<sub>maj</sub> × b<sub>min</sub> [arcsec<sup>2</sup>]) cm<sup>−2</sup> |
| HI mass | M<sub>HI</sub> = 2.356 × 10<sup>5</sup> × D<sup>2</sup> [Mpc] × S<sub>int</sub> [Jy km/s] solar masses; we use D = 4.9 Mpc for M83 |

{: .problem }
> **Spectra 97 times too bright.** SoFiA can write source spectra as the *sum* of Jy/beam values per channel. Plotted as Jy, they are too bright by the beam area in pixels (here 96.6). Our first plots showed M83 peaking at "380 Jy".
>
> **Cure:** check the units: the spectrum, added over all channels, should give the catalogue's `f_sum` divided by the beam area. `sofia_plots.py` checks this itself.

## The whole field

{% include fig.html src="16_sofia/sofia_mom0_column_density.png" caption="SoFiA moment 0 of the whole mosaic, as HI column density. The cyan numbers are SoFiA's source ids." %}

SoFiA found **190 sources**. Most are not separate galaxies:

- Many are **pieces of M83's own outer disk** (the numbers crowded along M83's arms). SoFiA joins voxels into one source only if they touch; faint parts of the outer arms with small gaps between them became separate "sources".
- Some are probably **noise**. In SoFiA 2.6 the **reliability filter** (which compares positive detections with the negative ones that pure noise also produces, and removes positive detections that look like noise) is **off** by default. The notes used SoFiA 2.1 defaults.

{: .pending }
> A second SoFiA run with the reliability filter on (`reliability.enable = true`) and a larger linking radius (`linker.radiusXY = 3`, `linker.radiusZ = 2`) would give a cleaner catalogue, with M83 as one source.

## M83: how much gas, and where

{% include fig.html src="16_sofia/sofia_M83_mom0.png" caption="M83: HI column density, with contours at 0.5, 1, 2, 5, 10 and 20 × 10<sup>20</sup> cm<sup>−2</sup>." %}

Compare this with the 3σ-clipped map of [step 15](../15-moments/): there are no noise speckles, and faint gas is kept, because SoFiA's mask is made in 3D and at several smoothing scales.

- **The inner disk** (about 10′ across) has the highest column densities, above 10<sup>21</sup> cm<sup>−2</sup>, with clear **spiral arms**.
- **A hole in the very centre**: partly HI **absorption** against the bright nucleus ([step 19](../19-hi-absorption/)), and partly real: in the dense centre most gas is **molecular**.
- **A ring and long arms** around the inner disk, at a few × 10<sup>20</sup> cm<sup>−2</sup>.
- **The outer arms** reach about **35–40′** from the centre (about 50–55 kpc at 4.9 Mpc), far beyond the optical galaxy. This "extended HI disk" is what M83 is famous for.

{: .problem }
> **Dark gaps that are too dark.** Some gaps between the arms are partly **negative bowls** from the imaging: uncleaned bright emission and the missing largest scales (MeerKAT's shortest baselines cannot see structures larger than about 20′). See the end of [step 19](../19-hi-absorption/). Real gaps exist, but these make them deeper.
>
> **Cure:** multi-scale clean of the cube with a mask, and adding single-dish (Parkes) data ("feathering").

## M83: how the gas moves

{% include fig.html src="16_sofia/sofia_M83_mom1_velocity.png" caption="M83: HI velocity field (moment 1), radio velocity in the LSRK frame, contours every 20 km/s." %}

- **Rotation:** the north-east side approaches us (blue, down to about 380 km/s), the south-west side recedes (red, up to about 650 km/s). The middle (white) is the **systemic velocity**, about 512 km/s.
- **The lines of equal velocity twist with radius.** In the inner disk they run north-west to south-east; further out they turn. The direction in which the disk rotates fastest (the **kinematic major axis**) changes with radius: the outer disk is **warped** (tilted differently from the inner disk), as known from earlier HI studies of M83.
- **Patches that do not follow the rotation**: some isolated dark red blobs in the south-east, and blue specks inside red regions in the south. These are gas at unusual velocities (perhaps clouds falling in, or extra-planar gas), or noise and imaging artefacts at the edges of the mask. They need checking in the cube before any interpretation.

## M83: how turbulent the gas is

{% include fig.html src="16_sofia/sofia_M83_mom2_dispersion.png" caption="M83: HI velocity dispersion (moment 2)." %}

- **Outer disk: about 5–12 km/s**, typical for the warm, turbulent HI of spiral galaxies.
- **Inner disk: about 15–25 km/s.** Higher, partly because of real turbulence from star formation, and partly because of **beam smearing**: within one beam (about 0.7 kpc), the rotation velocity changes quickly near the centre, which widens the spectrum.
- **Yellow patches (35 km/s and more)**, mostly in the south: places where the spectrum has **two components** at different velocities (for example the unusual-velocity gas seen in the velocity field, on top of the normal disk). Moment 2 of a double spectrum is large, but it is not turbulence.

## M83's global HI profile

{% include fig.html src="16_sofia/sofia_M83_global_profile.png" caption="M83's HI spectrum, added over SoFiA's mask in the primary-beam-corrected cube." %}

The source table at the end of this page gives the numbers:

- **Double-horned**, the signature of a rotating disk, and **asymmetric**: the receding side is brighter. M83 has more gas on one side (it is lopsided). Because of this, the flux-weighted velocity (520 km/s) differs from the middle of the profile (512 km/s).
- **W<sub>50</sub> = 261 km/s, W<sub>20</sub> = 293 km/s**: the width of the profile at 50% and 20% of its peak. With M83's inclination, they give its rotation speed.
- **Unlike step 15's clipped profile**, the profile goes to zero outside the line. The 650 mJy "clipping floor" of step 15 is gone, because SoFiA uses all voxels inside its mask (positive and negative), and none outside.
- **S<sub>int</sub> = 802 Jy km/s, M<sub>HI</sub> = 4.5 × 10<sup>9</sup> M<sub>☉</sub>** for D = 4.9 Mpc (M<sub>HI</sub> scales with D<sup>2</sup>).

{: .ours }
> The total flux is a **lower limit**: negative bowls inside the mask, and the smooth large-scale emission that MeerKAT cannot measure, both reduce it. A single-dish measurement (for example from the Parkes HIPASS survey) is the check: an interferometer can only reach the single-dish flux if nothing is missed.

## The neighbours

{% include fig.html src="16_sofia/sofia_brightest_sources.png" caption="The four brightest sources after M83: HI column density (left) and spectrum (right)." %}

| SoFiA id | RA (J2000) | Dec (J2000) | v<sub>sys</sub> (km/s, W<sub>50</sub> midpoint) | W<sub>20</sub> (km/s) | W<sub>50</sub> (km/s) | S<sub>int</sub> (Jy km/s) | M<sub>HI</sub> (M<sub>☉</sub>, D = 4.9 Mpc) |
|--:|:--|:--|--:|--:|--:|--:|--:|
| 2 | 13:37:07.0 | -29:53:48 | 512 | 293 | 261 | 802.0 | 4.5 × 10<sup>9</sup> |
| 27 | 13:41:37.1 | -29:54:48 | 479 | 55 | 37 | 20.1 | 1.1 × 10<sup>8</sup> |
| 142 | 13:36:31.4 | -29:14:06 | 574 | 47 | 31 | 5.2 | 3 × 10<sup>7</sup> |
| 153 | 13:40:18.6 | -28:53:34 | 581 | 42 | 30 | 4.3 | 2.4 × 10<sup>7</sup> |
| 156 | 13:39:17.9 | -30:03:23 | 586 | 57 | 34 | 2.1 | 1.2 × 10<sup>7</sup> |

- **id 2: M83 itself.**
- **id 27: NGC 5264**, a dwarf irregular galaxy near M83. Its position and velocity (about 480 km/s) match the catalogue. Its HI shows an inner ring or arc.
- **id 142: UGCA 365** (ESO 444-G078), another dwarf near M83, at about 575 km/s.
- **id 153**: a compact HI source at about 581 km/s, not yet identified. A search in the NASA/IPAC Extragalactic Database (NED) at its position will show whether it is a known dwarf galaxy.
- **id 156**: at the eastern edge of M83's outer disk, with a velocity close to that of M83's gas there. Probably a **piece of M83's outer disk** that SoFiA did not link, not a separate galaxy.

The masses in the table use M83's distance. Each dwarf has its own distance measurement, which may differ a little.

## HI and radio continuum together

{% include fig.html src="16_sofia/sofia_hi_on_continuum.png" caption="HI column density contours on the self-calibrated 1.4 GHz continuum mosaic (step 18)." %}

The radio continuum (from star formation: cosmic rays and hot gas) is concentrated in the bright inner disk, about 10′ across. The HI disk is many times larger. Most of M83's atomic gas lies far outside the region where stars are forming now: a large reservoir of gas for the future.
