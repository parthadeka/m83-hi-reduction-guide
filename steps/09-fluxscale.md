---
title: "9. Flux scale transfer"
parent: Reduction steps
nav_order: 9
permalink: /steps/09-fluxscale/
---

# Step 9. Flux scale transfer
{: .no_toc }

Pipeline stage: `fluxscale` · CASA task: `fluxscale`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

In step 8, the gain calibrator PKS1313-333 was solved against a **1 Jy** model, so its gain amplitudes are wrong by an unknown factor. This step finds that factor, and with it, **the true flux of PKS1313-333**.

## How it works

The key fact: **an antenna's gain does not depend on which source it looks at.** Antenna m005 amplifies the signal by the same amount whether it points at J1939-6342 or at PKS1313-333 (apart from slow changes with time).

- On J1939-6342, we used the correct model (14.95 Jy), so its gain amplitudes are **true**.
- On PKS1313-333, we used 1 Jy, so its gain amplitudes are **scaled** by √(S<sub>true</sub> / 1 Jy).

`fluxscale` compares the two sets of amplitudes, antenna by antenna, and averages:

> S<sub>true</sub>(PKS1313-333) = 1 Jy × (mean gain on PKS1313-333 / mean gain on J1939-6342)²

It then writes a **new** table (`fluxscale1`) with the PKS1313-333 amplitudes corrected. This new table is the one we apply to the target in step 10.

{: .plain }
> J1939-6342 is our ruler. In this step, we hold PKS1313-333 next to the ruler and read off its length. After that, PKS1313-333 can act as a ruler too, right next to M83.

## What we ran

```python
fluxscale(vis=msfile, caltable=gtable1, fluxtable=ftable1,
          reference='J1939-6342', transfer='PKS1313-333')
fluxscale(vis=msfile, caltable=gtable1, fluxtable=ftable1_alt,
          reference='J1939-6342', transfer='J1311-2216')
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `caltable` | `G1` | The table with all three sources from step 8. |
| `reference` | `'J1939-6342'` | The source with the correct model. |
| `transfer` | `'PKS1313-333'` | The source whose flux we want to find. |
| `fluxtable` | a new table | The corrected gains are written here. G1 itself is not changed. |

The second call does the same for the spare calibrator J1311-2216. Its table is kept but not applied.

## Result

```
PKS1313-333:  0.7241 ± 0.0037 Jy      (the notes found 0.7232 Jy)
```

This agrees with the notes to **0.1%**. It also agrees with the step 8 plots: the gain amplitudes on PKS1313-333 were about √0.72 ≈ 0.85 times those on J1939-6342.

The **error** (± 0.0037 Jy, or 0.5%) comes from the scatter between antennas. A small error like this means all antennas agree; the calibration is consistent.

{: .ours }
> A new reduction that finds the same calibrator flux as an older, independent reduction is a strong sign that both are correct.

## What can go wrong

{: .problem }
> **The flux is very different from expected values** (for example from the notes, a catalogue or the calibrator database).
>
> **Possible causes:** wrong flux model in step 4; the flux calibrator and gain calibrator observed at very different elevations (the antenna gain then really differs); bad antennas included; one source resolved (not a point source).
>
> **Cure:** check the `setjy` result; check the gain plots for outlier antennas and flag them; for a resolved calibrator, limit the baselines with `uvrange` in step 8.

{: .problem }
> **Large error bar** (several percent or more).
>
> **Cause:** antennas disagree. Often one or two antennas have bad solutions on one of the sources.
>
> **Cure:** look at the per-antenna ratios that `fluxscale` prints in the log. Flag the outliers and redo steps 8 and 9.

{: .problem }
> **Applying the wrong table.** If you apply `G1` instead of `fluxscale1` to the target, all fluxes in your images are wrong by the factor √(0.72) per antenna, so the target comes out 1/0.72 ≈ 1.4 times too bright.
>
> **Cure:** always apply the table written by `fluxscale`.

{: .note }
> Variable sources: many gain calibrators are quasars, and their brightness changes over months and years. That is why we measure PKS1313-333 again in every observation, instead of taking its flux from a catalogue.
