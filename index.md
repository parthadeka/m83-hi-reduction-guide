---
title: Start here
nav_order: 1
permalink: /
---

# Reducing MeerKAT HI data of M83, step by step
{: .fs-8 }

A guide for people who are reducing radio data for the first time. Every step is shown with the real plots from one reduction.
{: .fs-5 .fw-300 }

---

## What is this guide?

**M83** is a large spiral galaxy in the southern sky. It is about 15 million light years away. Like most spiral galaxies, it contains a lot of **neutral hydrogen gas** (we write it as **HI**, said "H-one"). Neutral hydrogen gives out radio waves at one special frequency: **1420.405752 MHz** (a wavelength of 21 cm). A radio telescope can "see" this gas, even where there are no stars.

On 8 December 2019, the **MeerKAT** radio telescope in South Africa observed M83 for about 11 hours. The telescope does not give us a picture directly. It gives us a large file of raw measurements. **Data reduction** is the work of turning these raw measurements into a clean, correct picture (an image) of the gas.

This guide goes through every step of that work. For each step it explains:

- **what the step does**, in simple words;
- **why we need it**;
- **which values (parameters) we chose, and why**;
- **what the plots show**, and how to read them;
- **what can go wrong**, how to notice it, and how to fix it.

Our dataset is quite clean and well behaved. That is good for learning, but it also means some problems never appear in our plots. So each page also describes problems that often happen in *other* datasets.

{: .plain }
> Think of the telescope as 60 separate microphones spread over 8 km of desert. Each pair of microphones "listens" to the sky together. Data reduction is like tuning, cleaning and combining all these recordings, so that together they act as one giant ear.

## How to use this guide

- Read the pages **in order**. Each step needs the result of the step before it.
- Words in **bold** are usually explained on the same page. All technical words are also collected in the [Glossary]({{ '/glossary/' | relative_url }}). Open it in a second browser tab.
- Click on any plot to see it at full size.
- Many plots exist once per antenna. The page shows one example. The full set is hidden in a box called "Show the plots for all antennas". Click the box to open it.

## The steps

| Step | Page | What happens |
|:-----|:-----|:-------------|
| 0 | [The observation]({{ '/observation/' | relative_url }}) | What was observed, and how |
| 1 | [Preparing the data]({{ '/steps/01-preparing-data/' | relative_url }}) | Cut out only the part of the data we need |
| 2 | [First look at the raw data]({{ '/steps/02-first-look/' | relative_url }}) | Look at the data before we change anything |
| 3 | [Flagging]({{ '/steps/03-flagging/' | relative_url }}) | Remove bad data |
| 4 | [Flux model]({{ '/steps/04-flux-model/' | relative_url }}) | Tell CASA how bright our reference source is |
| 5 | [Delay calibration]({{ '/steps/05-delay/' | relative_url }}) | Correct timing errors between antennas |
| 6 | [Initial phase calibration]({{ '/steps/06-initial-phase/' | relative_url }}) | A quick phase correction, needed for the next step |
| 7 | [Bandpass calibration]({{ '/steps/07-bandpass/' | relative_url }}) | Correct how each antenna responds at each frequency |
| 8 | [Gain calibration]({{ '/steps/08-gain/' | relative_url }}) | Correct how each antenna changes with time |
| 9 | [Flux scale transfer]({{ '/steps/09-fluxscale/' | relative_url }}) | Put everything on a true brightness scale (janskys) |
| 10 | [Apply and check the calibration]({{ '/steps/10-apply-and-check/' | relative_url }}) | Correct the target data and look for problems |
| 11 | [Split the target and expected noise]({{ '/steps/11-split-and-noise/' | relative_url }}) | Keep only M83, and predict how good the image can be |
| 12 | [Continuum subtraction]({{ '/steps/12-continuum-subtraction/' | relative_url }}) | Remove everything that is not the HI line |
| 13 | [Imaging: setup and noise test]({{ '/steps/13-imaging-setup/' | relative_url }}) | Make a first test image and measure the real noise |
| 14 | [The HI data cube]({{ '/steps/14-cube/' | relative_url }}) | Make the 3D image of the gas |
| 15 | [Moment maps]({{ '/steps/15-moments/' | relative_url }}) | Turn the 3D cube into 2D maps of gas amount, motion and speed spread |
| 16 | [Continuum image]({{ '/steps/16-continuum-image/' | relative_url }}) | An image of the radio continuum (not the HI line) |
| 17 | [Export and next steps]({{ '/steps/17-export-and-next/' | relative_url }}) | Save to FITS, and find the gas properly with SoFiA |
| 18 | [Continuum self-calibration]({{ '/steps/18-continuum-selfcal/' | relative_url }}) | Improve the calibration with the target itself, and make a 6 times deeper continuum mosaic |
| 19 | [HI absorption at the nucleus]({{ '/steps/19-hi-absorption/' | relative_url }}) | Cold gas in front of M83's nucleus, confirmed with long baselines |
| 20 | [SoFiA results]({{ '/steps/20-sofia-results/' | relative_url }}) | M83's HI maps, motions and total HI mass, and its neighbouring galaxies |

## The software

- **CASA** (Common Astronomy Software Applications), version **6.7.2**. CASA is the standard program for reducing radio interferometer data. The original reduction notes used the older CASA 5.6.2. A few task names changed between versions; the pages say where.
- The work ran on the **ilifu** computing cluster in Cape Town, using **slurm** (a system that queues and runs long jobs on the cluster's computers).
- Two Python scripts did steps 1 to 17: `m83_pipeline.py` (steps 1 to 12) and `m83_imaging.py` (steps 13 to 17).
- Step 18 uses **WSClean** (a fast imaging program) with CASA for the self-calibration, and **katbeam** (a model of the MeerKAT primary beam).

{: .note }
> The scripts follow a set of reduction notes written in December 2019 for this same observation. When this guide says "the notes", it means those notes. Most parameter values come from them. Where we changed something, the page says so and explains why.
