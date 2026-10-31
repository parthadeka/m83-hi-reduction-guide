---
title: "11. Split the target and expected noise"
parent: Reduction steps
nav_order: 11
permalink: /steps/11-split-and-noise/
---

# Step 11. Split the target, and predict the noise
{: .no_toc }

Pipeline stages: `split_target`, `noise` · CASA task: `split`, plus our own Python function
{: .step-meta }

1. TOC
{:toc}

---

## Part 1: split the target

### What it does

```python
split(vis=msfile, outputvis=vis_hi, field='M83_0,M83_1,M83_2,M83_3,M83_4,M83_5,M83_6',
      datacolumn='corrected')
```

This makes a **new MS that contains only the seven M83 pointings**, and only their **calibrated** data. In the new MS, the calibrated data are stored in the `DATA` column (the old `CORRECTED_DATA` becomes the new `DATA`).

| Parameter | Value | Why |
|:----------|:------|:----|
| `field` | all seven M83 pointings | We do not need the calibrators any more. All seven pointings stay together in one MS, because the mosaic imaging (step 13) uses them all at once. |
| `datacolumn` | `'corrected'` | Keep the calibrated data. |

We keep all 2001 channels here. The continuum fit in step 12 needs the line-free channels on both sides of the line.

The new MS is called `1575768670_sdp_l0_M83_0.ms`. The name only mentions `M83_0` because it was built from the first name in the list, but it contains **all seven** pointings.

### Why

- The target MS is smaller (about 400–450 GB, against 870 GB), so later steps run faster.
- The calibration is now "frozen" into the data. Later steps cannot accidentally use a wrong calibration.

{: .problem }
> **Splitting too early.** If you later find a calibration problem, you must apply it again and split again. Only split when you are happy with the calibration checks (step 10).

{: .problem }
> **Flags are copied, not linked.** Flags you add to the big MS *after* the split do not appear in the target MS. Flag the target MS directly, or split again.

## Part 2: the theoretical noise

### What it does

Before imaging, we **calculate** how much noise the images should have, from the properties of the telescope. Later we compare this prediction with the noise we actually measure. If they agree, the data are behaving well. If the measured noise is much larger, something is wrong.

### The radiometer equation

The noise in one channel of an interferometer image is:

> σ = (2 k T<sub>sys</sub>) / (η A √(N<sub>pol</sub> × Σ t Δν))

In words:

- **T<sub>sys</sub>** (system temperature): how much noise the receiver and the sky add. Lower is better.
- **η** (aperture efficiency): what fraction of the dish area really collects signal.
- **A**: the area of one dish (13.5 m diameter).
- **k**: Boltzmann's constant.
- **N<sub>pol</sub> = 2**: two polarisations (XX and YY).
- **Σ t Δν**: the total of (time × channel width) over all unflagged baselines. More time, more baselines and wider channels all reduce the noise.

{: .plain }
> Noise goes down with the square root of the observing time. To halve the noise, you need four times as long.

Our script uses two tables of T<sub>sys</sub>/η against frequency, both from the notes: an older 2019 table and a newer measured one. It counts the unflagged data exactly, field by field, from the `FLAG` column.

### The results

| Field | Time on source | Noise per channel (2019 table) | Noise per channel (newer table) |
|:------|:--------------:|:------------------------------:|:-------------------------------:|
| M83_0 | 55.0 min | 0.552 mJy | 0.497 mJy |
| M83_1 | 54.8 min | 0.553 mJy | 0.498 mJy |
| M83_2 | 54.1 min | 0.557 mJy | 0.501 mJy |
| M83_3 | 53.4 min | 0.560 mJy | 0.504 mJy |
| M83_4 | 53.6 min | 0.560 mJy | 0.503 mJy |
| M83_5 | 53.8 min | 0.558 mJy | 0.502 mJy |
| M83_6 | 53.6 min | 0.562 mJy | 0.505 mJy |
| PKS1313-333 | 67.0 min | 0.500 mJy | 0.450 mJy |
| J1939-6342 | 29.9 min | 0.756 mJy | 0.680 mJy |

So for **one M83 pointing**, we expect about **0.50 to 0.55 mJy per 26 kHz channel**. Where pointings overlap in the mosaic, the noise will be somewhat lower. Our results agree with the values in the notes (0.50–0.51 mJy with the newer table).

{% include fig.html src="11_theoretical_noise/theoretical_noise_vs_freq.png" caption="Theoretical noise per channel against frequency, for each field (2019 table). Solid lines: M83 pointings; dashed lines: calibrators." %}

**What we see:** the seven M83 pointings lie almost on top of each other (they had the same observing time), and the noise falls slightly towards higher frequency (the receiver is a little better there). The calibrators have higher noise because they were observed for less time (except PKS1313-333, which had more).

### What can go wrong

{: .problem }
> **The measured image noise is much higher than predicted** (more than about 20–30%).
>
> **Possible causes:** remaining RFI; bad antennas or baselines not flagged; calibration errors; sidelobes from bright sources (especially in continuum images); continuum not removed properly.
>
> **Cure:** find which of these it is (look at the corrected data again, image line-free channels, check per antenna) and fix it.

{: .problem }
> **The measured noise is lower than predicted.** Usually a sign that weights or the flagging count are wrong, or that the T<sub>sys</sub> table is too pessimistic.
