---
title: "19. HI absorption at the nucleus"
parent: Reduction steps
nav_order: 19
permalink: /steps/19-hi-absorption/
---

# Step 19. HI absorption against M83's nucleus
{: .no_toc }

Script: `m83_hi_absorption.py` · CASA task: `tclean` (cubes with a minimum baseline length) · Viewer: CARTA
{: .step-meta }

1. TOC
{:toc}

---

## How we found it: a hole in the HI map

The SoFiA moment 0 map (the total amount of HI) has a small dark hole exactly in the centre of M83, about one beam across. It sits on the brightest point of the continuum image: M83's nucleus.

{% include fig.html src="15_hi_absorption/01_continuum_vs_mom0_centre.png" caption="Left: the self-calibrated continuum (step 18) around the centre of M83, with the bright nucleus. Right: the SoFiA moment 0 map of HI (primary-beam corrected), same area. The dark spot in the HI map is exactly on the continuum nucleus." %}

Two different things are visible in the HI map:

- **The small dark spot on the nucleus**: this page is about it.
- **A wider area of low HI** (about 1–2′ across) around it, inside the bright HI ring and arms. This is most likely real, and common in barred spirals: in the dense centre, most gas is **molecular** hydrogen (H<sub>2</sub>), which does not emit at 21 cm. M83's strong bar drives gas into the centre, where it becomes molecular and forms stars.

## What is HI absorption?

Normally HI **emits** at 1420 MHz. But cold HI in front of a bright **continuum** source also **absorbs** a little of that source's radio light at the same frequency. After continuum subtraction, absorption appears as **negative** values in the line channels. In a moment 0 map it cancels part of the emission, and can make a hole.

How much is absorbed is described by the **optical depth** τ (tau): the fraction of light that gets through is e<sup>−τ</sup>. If S<sub>c</sub> is the continuum brightness and S the brightness in a line channel (before continuum subtraction):

  τ(v) = −ln(S / S<sub>c</sub>)

From τ we get the **column density** of the absorbing gas:

  N<sub>HI</sub> = 1.823 × 10<sup>18</sup> × T<sub>s</sub> × ∫τ dv  cm<sup>−2</sup>

where ∫τ dv is τ added up over the line (in km/s) and T<sub>s</sub> is the **spin temperature** (a measure of how cold the gas is; we use 100 K, a common assumption, and N<sub>HI</sub> scales with it).

{: .plain }
> A lamp behind a coloured glass: the glass (the cold gas) takes away a little light of one colour (1420 MHz). How much it takes away tells us how much glass there is.

## Step 1: the spectrum, three tests

Spectra in CARTA from the robust 1 cubes (beam 29″ × 26″):

{% include fig.html src="15_hi_absorption/02_spectrum_nucleus_contsub_29arcsec.png" caption="Continuum-subtracted cube, at the nucleus. The spectrum is flat at zero away from the line, and has a negative trough from about 1.4175 to 1.4185 GHz (M83's HI velocities)." %}

**Test 1: is it a continuum subtraction error?** No. A continuum error shifts *all* channels. Here the line-free channels are at zero on both sides; only M83's HI channels are negative.

{% include fig.html src="15_hi_absorption/04_spectrum_nucleus_no_contsub_29arcsec.png" caption="Cube *before* continuum subtraction, at the nucleus: the continuum is about 248 mJy/beam, with a dip of up to about 6–7 mJy (2.5%) in the HI channels." %}

**Test 2: is there really less light than the continuum?** Yes. Before continuum subtraction, the nucleus spectrum dips **below** the continuum level. Emission can only add light, so something takes light away.

{% include fig.html src="15_hi_absorption/03_spectrum_off_nucleus_contsub_29arcsec.png" caption="Continuum-subtracted cube, about 52″ from the nucleus: positive HI emission near 500 km/s, but negative wings on both sides." %}

**Test 3: does it only happen on the continuum?** Not clearly. About 52″ away, there are negative wings too. These could be absorption against the fainter extended continuum of the starburst region, *or* a **negative bowl**: the negative sidelobes of bright, uncleaned HI emission nearby, which appear at exactly the velocities of that emission. With a 29″ beam, the two cannot be separated.

## Step 2: the decisive test, long baselines only

**Short baselines** see large, smooth structures; **long baselines** only see small structures. If we make images with only the long baselines, smooth HI emission and bowls made from it disappear, but a compact continuum source and any absorption against it stay. So:

- if the dip is **absorption against a compact source**, its depth in mJy stays the same as short baselines are removed;
- if it is a **bowl** or emission effect, it fades.

`m83_hi_absorption.py` makes small cubes (256 × 2″) around the nucleus from the data *before* continuum subtraction, central pointing only, with four minimum baseline lengths:

```python
tclean(vis=vis, field='M83_4', uvrange=uvrange, datacolumn='data', specmode='cube',
       start=1430, nchan=161, width=1, outframe='LSRK', phasecenter=NUCLEUS,
       imsize=256, cell='2arcsec', weighting='briggs', robust=0.0, niter=50000, nsigma=3.0,
       usemask='pb', pbmask=0.5, restoringbeam='common', parallel=True)
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `uvrange` | `''`, `'>2klambda'`, `'>5klambda'`, `'>10klambda'` | Minimum baseline length, in thousands of wavelengths. 5 kλ (about 1 km at 21 cm) removes structures larger than about 40″. |
| `datacolumn` | `'data'` of the MS before continuum subtraction | We need the continuum level S<sub>c</sub> to calculate τ |
| `start`, `nchan` | MS channels 1430–1590 | M83's HI (about 355–660 km/s) with line-free channels on both sides |
| `robust` | 0 | A smaller beam than robust 1, so less extended continuum and emission in the beam |
| `restoringbeam` | `'common'` | One beam for all channels (see "what can go wrong") |

Then, for each cube, the script takes the spectrum at the continuum peak, fits a straight line to the line-free channels (the continuum), and calculates τ(v), ∫τ dv and N<sub>HI</sub>.

{% include fig.html src="15_hi_absorption/05_nucleus_robust0_channel.png" caption="One channel of the all-baselines robust 0 cube (beam 8.6″ × 7.5″): the compact continuum nucleus (cyan marker)." %}

## The result

{% include fig.html src="15_hi_absorption/06_nucleus_absorption_by_uvrange.png" caption="Top: the spectrum at the nucleus divided by the continuum, for each minimum baseline length. Bottom: the optical depth τ. Grey: the channels where M83's HI can be." %}

| Baselines | Beam | Continuum S<sub>c</sub> | Deepest dip | Dip / S<sub>c</sub> | Peak τ | ∫τ dv (km/s) | N<sub>HI</sub> (T<sub>s</sub> = 100 K) |
|:--|:--|--:|--:|--:|--:|--:|--:|
| all | 8.6″ × 7.5″ | 80.3 mJy | 9.0 mJy (8.7σ) | 11% | 0.12 | 9.3 ± 0.6 | 1.7 × 10<sup>21</sup> cm<sup>−2</sup> |
| > 2 kλ | 7.6″ × 6.6″ | 66.3 mJy | 9.0 mJy (8.0σ) | 14% | 0.15 | 11.4 ± 0.8 | 2.1 × 10<sup>21</sup> |
| > 5 kλ | 6.7″ × 5.9″ | 49.8 mJy | 9.1 mJy (6.6σ) | 18% | 0.20 | 15.1 ± 1.3 | 2.8 × 10<sup>21</sup> |
| > 10 kλ | 6.0″ × 5.2″ | 30.8 mJy | 9.6 mJy (4.9σ) | 31% | 0.37 | 20.8 ± 3.1 | 3.8 × 10<sup>21</sup> |

**This confirms real HI absorption:**

- As short baselines are removed, the continuum per beam drops from 80 to 31 mJy: the extended continuum of the starburst region is "resolved out".
- But the absorbed flux stays the same, **about 9–9.6 mJy**, at every cut. All the absorption happens against a **compact** continuum source, which every beam contains completely: smaller than about 5″, or about **120 pc** (at M83's distance, 1″ ≈ 24 pc).
- A bowl or emission effect would fade with long baselines. This does not.

**What the numbers mean:**

- τ grows with resolution only because each beam contains less of the extended continuum, which is *not* absorbed. Even at > 10 kλ some of it remains, so **τ ≥ 0.37 and N<sub>HI</sub> ≥ 3.8 × 10<sup>21</sup> (T<sub>s</sub>/100 K) cm<sup>−2</sup> are lower limits** for the gas in front of the compact source.
- The absorption is centred at about **515–520 km/s**, at M83's **systemic velocity** (about 513 km/s), and is about **100 km/s** wide (about 470–570 km/s). Absorbing gas is always *in front* of the source, so a shift to higher velocities would mean gas falling in, and a shift to lower velocities gas flowing out. We see no clear shift: the gas is most likely **rotating close to the nucleus** (a circumnuclear disk or the nuclear starburst ring), seen in front of the compact source.
- The noise grows as baselines are removed (1.0 → 2.0 mJy per channel). The > 5 kλ spectrum is the best compromise: τ<sub>peak</sub> ≈ 0.20, ∫τ dv = 15.1 ± 1.3 km/s.

{: .ours }
> In the 29″ cube, the absorption looked shifted to higher velocities, which suggested gas falling in. At higher resolution this disappeared: HI **emission** in the large beam had filled in part of the absorption and changed its shape. Always measure absorption at the highest resolution the data allow.

## What it could be used for

A quick literature search found no published HI 21 cm absorption study of M83's nucleus (optical absorption by Na I and Ca II towards the nucleus, spread over about 200 km/s, is known). A full literature search is needed before claiming it is new. Possible science:

- **Cold gas in a starburst nucleus**: τ and N<sub>HI</sub>, and with the HI emission at the same position, the spin temperature of the gas.
- **Gas motions near the nucleus**: compare the absorption velocities with the rotation of the gas (HI moment 1, or CO maps), to test for inflow or outflow along M83's bar.
- **Which compact source?** M83's nucleus is complex: the optical nucleus, the kinematic centre and a bright arc of young star clusters are a few arcseconds apart. Images at about 1″ resolution (for example the VLA in A configuration) would show which one is behind the absorbing gas.

## How to make it better

- **Apply the continuum self-calibration gains** ([step 18](../18-continuum-selfcal/)) to the HI data before imaging: lower noise, especially for the long-baseline spectra.
- **Better velocity resolution**: 5.5 km/s channels are enough for a 100 km/s line, but not for narrow components of cold clouds. MeerKAT's narrow-band (zoom) modes give channels below 1 km/s.

## What can go wrong

{: .problem }
> **A bowl that looks like absorption.** Around bright extended HI emission, uncleaned emission leaves negative sidelobes at the same velocities. At low resolution this looks just like absorption, even before continuum subtraction.
>
> **Cure:** the long-baseline test on this page: absorption against a compact source keeps its depth in mJy; a bowl fades.

{: .problem }
> **One beam per channel.** Without `restoringbeam='common'`, `tclean` gives each channel of a cube its own beam. `ia.restoringbeam()` then returns a table of beams instead of one, and code that expects one beam crashes (our first run did, with `KeyError: 'major'`).
>
> **Cure:** use `restoringbeam='common'` for cubes, or read the beam of one channel from the table.

{: .problem }
> **Statistic "Sum" in CARTA's spectral profiler.** For a region of more than one pixel, "Sum" adds pixels and is not in Jy/beam.
>
> **Cure:** use a point region, or the statistic "Mean".
