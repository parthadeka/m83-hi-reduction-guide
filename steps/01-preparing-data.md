---
title: "1. Preparing the data"
parent: Reduction steps
nav_order: 1
permalink: /steps/01-preparing-data/
---

# Step 1. Preparing the data
{: .no_toc }

Pipeline stage: `split_subset` · CASA task: `mstransform`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

The raw data file contains all 32768 frequency channels and all four polarisations. We only need a small part of it: the channels around the HI line of M83, and the XX and YY polarisations. This step **copies only that part into a new, smaller file**. All later steps work on the small file.

The data are stored in a format called a **Measurement Set** (MS). An MS is not one file. It is a folder with many tables inside. The main table has one row for each baseline and each time step. Its columns hold the visibilities and information about them.

## Why we need it

- **Speed.** Every later step reads the whole MS, often many times. Fewer channels means less to read.
- **Disk space.** The full-band data would need many terabytes on disk once calibrated.
- **Same channel numbers as the notes.** The 2019 notes cut the data the same way, so every channel number in the notes still means the same frequency here.

## What we ran

```python
mstransform(vis=RAW_MS, outputvis=msfile, datacolumn='data',
            spw='0:20000~22000', correlation='XX,YY')
```

| Parameter | Value | Why |
|:----------|:------|:----|
| `spw` | `'0:20000~22000'` | **spw** means **spectral window**: one block of frequency channels. Our data has only one, number `0`. After the colon come the channels to keep: 20000 to 22000. That is **2001 channels, from 1378.5 to 1430.7 MHz**. M83's HI (near 1418 MHz) sits in the middle, with room on both sides. The notes used exactly this cut. |
| `correlation` | `'XX,YY'` | Keep only the parallel hands. HI is not polarised, so XY and YX add nothing for us, and dropping them halves the data size. |
| `datacolumn` | `'data'` | Copy the raw, uncalibrated visibilities (the `DATA` column). |

After this cut, **channel 0 of the new file is channel 20000 of the original**. From now on, all channel numbers refer to the new file.

### The channel ranges we use later

These three ranges come from the notes. They are used again and again in later steps:

| Name in the script | Channels | Frequencies | Meaning |
|:-------------------|:---------|:------------|:--------|
| `valid_chans` | 1250 – 1800 | 1411.1 – 1425.5 MHz | The "good" part of the band that we calibrate and image |
| `tar_chans` | 1420 – 1580 | 1415.5 – 1419.6 MHz | Where M83's HI line is (with a safety margin) |
| `ref_chans` | 1250 – 1420 and 1580 – 1800 | both sides of the line | **Line-free** channels: only continuum, no HI. Used to measure and remove the continuum (step 12) |

## How long it took

This was the slowest step of the whole reduction: **about 38 hours**. `mstransform` works on one CPU core and must read the entire raw dataset (all 32768 channels, four polarisations) from disk to find the 2001 channels we want. It is limited by how fast the disk can be read (we say it is **I/O bound**), so adding more CPU cores would not have helped much.

The new MS was about **500 GB**. It is still large, because each visibility also has a **weight** and a **sigma** (an error estimate) per channel, stored in the `WEIGHT_SPECTRUM` and `SIGMA_SPECTRUM` columns. After calibration (step 10) a second data column is added, and the MS grows to about **870 GB**.

## What can go wrong

{: .problem }
> **Wrong channels selected.** If you copy channel numbers from someone else's notes, but your data was cut differently (or not cut at all), every channel range later in the reduction will point to the wrong frequencies. The continuum fit may then include the HI line, and the line will be partly removed.
>
> **How to notice:** run `listobs` on your new MS and check the first channel frequency and channel width. Here it should be 1378.46 MHz and 26.1 kHz.
>
> **Cure:** recompute the channel numbers from frequencies. Channel = (frequency − first frequency) / channel width.

{: .problem }
> **The job seems "stuck".** `mstransform` prints almost nothing while it works. On a big dataset it can be silent for many hours.
>
> **How to notice:** check whether the output MS folder is still growing (`du -sh` on it every hour or so).
>
> **Cure:** usually nothing; just wait. Ask for enough time in your slurm job (we asked for 4 days).

{: .problem }
> **Running out of disk space.** Calibration adds a full copy of the data (the `CORRECTED_DATA` column). Continuum subtraction makes another MS. Images and cubes add more.
>
> **Cure:** estimate the total before you start. For this reduction the whole chain needed about **2 to 2.5 TB**.

{: .note }
> The original notes created the MS directly from the MeerKAT archive with `mvftoms.py -C 20000,22000`. We started from a full-band MS that was already on the cluster, so we made the same cut with `mstransform` instead. The result is the same.
