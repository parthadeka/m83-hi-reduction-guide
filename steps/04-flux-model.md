---
title: "4. Flux model"
parent: Reduction steps
nav_order: 4
permalink: /steps/04-flux-model/
---

# Step 4. Setting the flux model
{: .no_toc }

Pipeline stage: `setjy` · CASA tasks: `clearcal`, `setjy`
{: .step-meta }

1. TOC
{:toc}

---

## The idea of calibration

Before the details, here is the big idea behind steps 4 to 9.

The telescope does not measure the sky perfectly. Each antenna changes the signal a little: it amplifies it by some amount and delays it by some amount, and both change with frequency and with time. We call these changes the **gains** of the antenna. A gain is a complex number:

- its **amplitude** says how much the antenna amplifies the signal;
- its **phase** says how much it shifts the signal in time.

For a baseline between antennas *i* and *j*:

> measured visibility = gain<sub>i</sub> × gain<sub>j</sub>* × true visibility

(the * means "complex conjugate", which here simply means "use the opposite phase").

**Calibration** finds the gains, so that we can divide them out. To find them, we look at a **calibrator**: a source whose true visibility we already know. Then:

> gain<sub>i</sub> × gain<sub>j</sub>* = measured visibility / true visibility

With 1770 baselines and only 60 unknown antenna gains, CASA can solve this very well. Each calibration table we make in steps 5–8 describes one *part* of the gains:

| Step | Table | What part of the gain | Changes with |
|:-----|:------|:----------------------|:-------------|
| 5 | **K** (delay) | A phase that grows linearly with frequency | Almost nothing |
| 7 | **B** (bandpass) | Amplitude and phase, channel by channel | Frequency |
| 8 | **G** (gain) | Amplitude and phase, one value per scan | Time |
| 9 | flux scale | Corrects the G amplitudes of the gain calibrator | — |

## What this step does

This step tells CASA the **true brightness** of our flux calibrator, J1939-6342, at every frequency. CASA writes this "model" into the `MODEL_DATA` column of the MS. Later, calibration tasks compare the measured data with this model.

First we run `clearcal`. It resets the `MODEL_DATA` column (and creates `CORRECTED_DATA`), so that no old model or old correction from a previous attempt is left in the file.

## What we ran

```python
clearcal(vis=msfile)
setjy(vis=msfile, field='J1939-6342', fluxdensity=-1, standard='Stevens-Reynolds 2016')
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `field` | `'J1939-6342'` | Our flux calibrator. |
| `standard` | `'Stevens-Reynolds 2016'` | A **flux standard** is a published table of the brightness of well-known calibrators against frequency. Stevens & Reynolds (2016) measured J1939-6342 accurately for southern telescopes. The notes used the same standard. If this standard had failed, the script would try the older `'Perley-Butler 2010'`. |
| `fluxdensity` | `-1` | "Take the flux density from the standard", instead of a number we type in. |

## Result

```
setjy Stevens-Reynolds 2016: J1939-6342 = 14.9506 Jy
```

So the model says J1939-6342 has **14.95 Jy** at our frequency. The model also includes the **spectral index** (how the brightness changes across the band). There is no plot for this step. You can check the model later: after calibration, J1939-6342 must come out at about 14.9 Jy (see [step 10](../10-apply-and-check/)).

{: .plain }
> J1939-6342 is our "ruler". Every brightness we measure in M83 is, in the end, a comparison with this one source. If the ruler is wrong, every number in the final results is wrong by the same factor.

## Why a point-source model is fine here

The model treats J1939-6342 as a **point source**: all its light comes from one point at the field centre. For a point source at the phase centre, the true visibility has the same amplitude on every baseline and zero phase. This makes the calibration simple and robust. J1939-6342 is small compared with our resolution, so this is a good approximation.

## What can go wrong

{: .problem }
> **The calibrator is not in the standard's list.** `setjy` then gives 0 Jy or an error, and every later step fails or gives nonsense.
>
> **How to notice:** read the `setjy` output. It must give a sensible, non-zero flux.
>
> **Cure:** choose another standard that contains the source, or give the flux with `standard='manual'`, `fluxdensity=[S, 0, 0, 0]` and `spix=` (the spectral index) and `reffreq=`.

{: .problem }
> **Wrong field name.** A typo in the name means the model is not set for the source you calibrate on. Calibration then compares the data with a model of 1 Jy (the default), and all fluxes come out wrong.
>
> **Cure:** copy the field names from `listobs`.

{: .problem }
> **An extended or resolved calibrator.** Some calibrators (for example 3C286 at higher resolution, or 3C138) are not point sources. A point model is then wrong on long baselines.
>
> **Cure:** use a model image of the calibrator (the `model=` parameter of `setjy`), or limit the calibration to short baselines with `uvrange`.

{: .problem }
> **Old models or corrections left in the MS.** If you rerun calibration without `clearcal`, an old `MODEL_DATA` may be used.
>
> **Cure:** run `clearcal` before `setjy`, as we did. Be careful: `clearcal` also resets `CORRECTED_DATA`, so only run it when you really want to start the calibration again.
