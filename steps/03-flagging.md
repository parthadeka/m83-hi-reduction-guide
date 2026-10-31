---
title: "3. Flagging"
parent: Reduction steps
nav_order: 3
permalink: /steps/03-flagging/
---

# Step 3. Flagging
{: .no_toc }

Pipeline stage: `flag` · CASA tasks: `flagdata`, `flagmanager`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

**Flagging** means marking data as "bad, do not use". The data are not deleted. Each visibility has a **flag** (a true/false value in the `FLAG` column). All CASA tasks skip flagged data. A flag can be removed again later.

## Why we need it

Bad data can ruin calibration and images. For example:

- a broken antenna gives wrong gain solutions, and the errors spread to every image pixel;
- RFI can be thousands of times brighter than the HI we want to see;
- absorption lines in a calibrator's spectrum make the calibration think the *receiver* has a dip at that frequency. That false dip would then be copied into the target data.

## What we flagged, and why

| Command | What it removes | Why |
|:--------|:----------------|:----|
| `flagdata(mode='manual', autocorr=True)` | **Autocorrelations** | An autocorrelation is an antenna combined with *itself* (for example m005 × m005). It measures the total power, not the sky structure. Imaging does not use it, and it is much noisier. |
| `flagdata(mode='manual', field='J1939-6342', spw='0:1594~1606')` | Channels 1594–1606 on the flux and bandpass calibrator | Milky Way HI absorption (seen in [step 2](../02-first-look/)). |
| `flagdata(mode='manual', field='PKS1313-333', spw='0:1608~1613')` | Channels 1608–1613 on the gain calibrator | Milky Way HI absorption in the direction of PKS1313-333. |
| `flagdata(mode='shadow')` | Shadowed data | When the source is low in the sky, one dish can block the view of another dish behind it. This is **shadowing**. Shadowed data have wrong amplitudes. CASA calculates it from the antenna positions. |
| `flagdata(mode='manual', antenna='m003')` | All data of antenna **m003** | m003 behaved differently from all other antennas in step 2. |

We do **not** flag the Milky Way HI channels on M83. Those channels (around 1600) are outside M83's HI range (1488–1538), so they cannot hurt the M83 line. They are, however, inside the continuum-fit range. We come back to this in [step 12](../12-continuum-subtraction/).

### Saving flag versions

```python
flagmanager(vis=msfile, mode='save', versionname='before_manual_flags')
...
flagmanager(vis=msfile, mode='save', versionname='after_manual_flags')
```

`flagmanager` saves a copy of all flags under a name. If we make a mistake, we can go back with `mode='restore'`. Our script restores `before_manual_flags` automatically each time the flag stage is run again, so flags never pile up from repeated runs.

{: .plain }
> Always save your flags before you change them. Flagging is cheap; un-flagging without a saved version is impossible.

## The plots

{% include fig.html src="03_flagging/flagged_fraction_per_antenna_before.png" caption="Fraction of flagged data per antenna, before this step." %}

{% include fig.html src="03_flagging/flagged_fraction_per_antenna_after.png" caption="Fraction of flagged data per antenna, after this step." %}

**What we see:**

- **Before:** nothing was flagged yet (0% for all antennas).
- **After:**
  - **m003 is 100% flagged**, as planned.
  - **Every antenna is about 1.7% flagged.** This is mostly the autocorrelations. Each antenna appears in 60 rows per time step (59 with other antennas, 1 with itself), and 1/60 is about 1.7%. The small calibrator channel flags add a little.
  - **m004 is about 3.5% flagged.** The extra part is most likely **shadowing**, because shadowing is the only flag in our list that removes data for some antennas and not others. In [step 5](../05-delay/) you will see that m004 has no solution for the first J1939-6342 scan, so that scan was flagged for m004.

In total, only a few percent of the data was flagged. This is a very clean dataset. A typical MeerKAT L-band dataset loses **30–50%** to RFI over the full band, but most of that RFI is outside our narrow range around 1420 MHz.

## What we did not do: automatic flagging

CASA has automatic flagging methods that look for outliers:

- **`tfcrop`**: fits smooth curves in time and frequency, and flags points far from them.
- **`rflag`**: flags points whose local scatter is too large.

The notes flagged only by hand, after looking at the plots. Our script has an option (`DO_AUTOFLAG`) to also run `tfcrop` on the calibrators, but we left it off, to stay close to the notes. For this clean dataset the hand flags were enough.

## What can go wrong

{: .problem }
> **Too little flagging.** Remaining RFI or bad antennas show up later as strange gain solutions (steps 6–8), spikes in the corrected data (step 10), or stripes and rings in the image.
>
> **Cure:** go back, flag, and redo the calibration from the start. We did find one remaining problem later: a short, very bright spike in one target scan around 07:03 UTC ([step 10](../10-apply-and-check/)).

{: .problem }
> **Too much flagging.** Automatic flaggers can flag real signal, for example a bright source or a strong HI line, because it looks like an outlier.
>
> **How to notice:** the flagged fraction is high in the channels where the HI line is.
>
> **Cure:** run automatic flagging only on calibrators, or on the target with the line channels excluded. Use a less strict threshold.

{: .problem }
> **Calibrator absorption lines not flagged.** The bandpass solution (step 7) will contain a false dip, which is then applied to the target. In the final cube this appears as a false absorption or emission feature at the same frequency in every pixel.
>
> **Cure:** look at the calibrators' spectra in step 2 and flag every line.

{: .problem }
> **Flags pile up when you rerun.** If you run the flagging step twice without restoring, the second run cannot "undo" anything.
>
> **Cure:** use `flagmanager` to save and restore versions, as our script does.
