---
title: "2. First look at the raw data"
parent: Reduction steps
nav_order: 2
permalink: /steps/02-first-look/
---

# Step 2. First look at the raw data
{: .no_toc }

Pipeline stage: `inspect_raw` · CASA tasks: `plotms`, `plotants`
{: .step-meta }

1. TOC
{:toc}

---

## What this step does

We make plots of the **raw** (uncalibrated) data, before we change anything. We look for:

- antennas that behave differently from the others,
- **RFI** (radio frequency interference: man-made radio signals, for example from satellites, aircraft or mobile phones),
- times when something went wrong,
- the general shape of the data, so that later we can see whether calibration improved it.

We also plot where the antennas are, and the **uv coverage** (explained below).

## Why we need it

Calibration assumes the data are mostly good. A single broken antenna or a strong RFI signal can spoil the calibration of *all* antennas. It is much easier to find problems now than to find them later in a bad image.

{: .plain }
> Never calibrate data you have not looked at. Ten minutes of plotting can save days of work.

## About the units in raw data

Raw visibilities are **not in janskys yet**. Their amplitude is in arbitrary "correlator units". We do not know the true brightness until after calibration. So in this step we compare antennas and times *with each other*, not with real numbers.

## The plots

All plots in this step are made with **plotms**, CASA's plotting tool. Some settings appear in many plots:

- `avgtime='1e8'` with `avgscan=True`: average over all times (1e8 seconds is "longer than the whole observation"), so each channel gets one point.
- `avgchannel='400'`: average 400 channels together, so each time step gets one point per 400 channels.
- `iteraxis='antenna'`: make one small panel per antenna. Each panel shows all the baselines of that antenna.
- `coloraxis='corr'`: colour the points by polarisation (XX in one colour, YY in another).

### Overview: every field, amplitude against frequency

{% include fig.html src="01_raw_data_inspection/all_fields_amp_vs_freq_overview_part1.png" caption="Raw amplitude against frequency for the first six fields. All 2001 channels, averaged over all times and all baselines." %}

{% include fig.html src="01_raw_data_inspection/all_fields_amp_vs_freq_overview_part2.png" caption="The same for the other six fields." %}

**How to read it.** Each panel is one field. The x-axis is frequency (1378 to 1431 MHz). The y-axis is the raw amplitude. The data are averaged over all baselines and the whole observation.

**What we see in our data:**

- **The calibrators are bright and smooth.** J1939-6342, J0408-6545, J1331+3030 and J1311-2216 show two clean lines. The two lines are **XX and YY**. They are at different heights because the X and Y receivers have different gains (amplification). Calibration will fix this.
- **Brightness falls slowly with frequency.** This is normal: most radio sources are brighter at lower frequencies. The rate of this change is called the **spectral index**.
- **A narrow dip near 1.420 GHz** on J1939-6342 and J0408-6545. This is **hydrogen in our own Milky Way**. Cold Milky Way gas lies between us and the calibrator and absorbs a little of its light at 1420.4 MHz. We will flag these channels on the calibrators in [step 3](../03-flagging/).
- **A spike near 1.381 GHz** in several M83 fields. This is RFI, very probably from the **GPS L3** satellite signal at 1381.05 MHz. It falls outside the range we use (1411–1425 MHz), so we can ignore it.
- **The M83 fields are faint** (a few thousandths of a unit), except **M83_4**, which is centred on the galaxy and shows its bright continuum.

{: .note }
> Why does averaging over all baselines make most of the target fields look so faint? Averaging complex numbers keeps only the signal at the **centre of each field**. Signals from other places have phases that change from baseline to baseline, and they cancel. M83_4 is centred on M83's bright nucleus, so its signal survives.

### Antenna amplitude against time

{% include fig.html src="01_raw_data_inspection/cals_amp_vs_time_per_antenna_m000-m006.png" caption="Raw amplitude against time for antennas m000 to m006. Both calibrators (PKS1313-333 and J1939-6342) are shown. Colours are baselines." %}

{% include gallery.html match="01_raw_data_inspection/cals_amp_vs_time" %}

**How to read it.** Each panel is one antenna. Each point is the amplitude of one baseline, averaged over 400 channels. The low points near 0.5 that repeat all night are the gain calibrator PKS1313-333 (about 0.7 Jy). The three tall groups at about 05:50, 08:20 and 10:45 are J1939-6342 (about 15 Jy).

**What we see:**

- Most antennas look the same: flat, stable, with no jumps.
- **m003 is different.** Its J1939-6342 amplitudes are lower than those of the other antennas and *rise* from one visit to the next (from about 3–5 to about 4–8). A healthy antenna should give the same answer each time it looks at the same steady source. This is the reason we flag m003 in step 3.
- The spread of points within each group (from about 3 to 11 units) comes from the different baselines. Each baseline has slightly different gains, and some see more of the other sources in the field. This spread is normal before calibration.

### Flux calibrator: amplitude against channel, and against time

{% include fig.html src="01_raw_data_inspection/fluxcal_amp_vs_chan_per_antenna_m000-m006.png" caption="J1939-6342: raw amplitude against channel for antennas m000 to m006, averaged over time. Colours: XX and YY." %}

{% include gallery.html match="01_raw_data_inspection/fluxcal_amp_vs_chan" %}

{% include fig.html src="01_raw_data_inspection/fluxcal_amp_vs_time_per_antenna_m000-m006.png" caption="J1939-6342: raw amplitude against time, one panel per antenna." %}

{% include gallery.html match="01_raw_data_inspection/fluxcal_amp_vs_time" %}

**What we see:**

- Each horizontal band is one baseline. The bands are almost flat across channels 1250–1800. This shows the receiver response (the **bandpass**) is smooth.
- **A dip at channel ~1600** on every antenna. This is the Milky Way HI absorption again (channel 1600 is 1420.3 MHz).
- m003 is again lower than its neighbours, and its amplitude grows with time.

### Gain calibrator: amplitude against channel

{% include fig.html src="01_raw_data_inspection/phasecal_amp_vs_chan_per_antenna_m000-m006.png" caption="PKS1313-333: raw amplitude against channel for antennas m000 to m006." %}

{% include gallery.html match="01_raw_data_inspection/phasecal_amp_vs_chan" %}

The bands are flat. There is a small dip near channel **1610**: Milky Way HI again, at a slightly different velocity because PKS1313-333 is in a different direction. We flag channels 1608–1613 on this source in step 3.

### Where the antennas are

{% include fig.html src="02_array_and_uv_coverage/antenna_positions.png" caption="Antenna positions in metres, relative to the array centre." %}

{% include fig.html src="02_array_and_uv_coverage/antenna_positions_log.png" caption="The same, with a logarithmic distance scale, so the dense core can be seen." %}

MeerKAT has a **dense core**: about 70% of the antennas are inside 1 km. The rest are spread out to about 8 km. This design gives:

- many **short baselines** (core antennas): good sensitivity to large, faint gas, which is exactly what HI studies need;
- some **long baselines** (outer antennas): the ability to see small details.

The **reference antenna**, `m059`, is one of the outer antennas. We explain the reference antenna in [step 5](../05-delay/).

### uv coverage

{% include fig.html src="02_array_and_uv_coverage/uv_coverage_M83_4_chan1440.png" caption="uv coverage of the central pointing M83_4, for one channel (1440). Axes are in wavelengths." %}

**What is uv coverage?** Each baseline, seen from the source, has a length and a direction. We call its two components **u** and **v** (measured in wavelengths). As the Earth rotates, every baseline traces an arc in the uv plane. The plot shows every (u, v) point we measured. This is how the interferometer "fills in" the big virtual dish over the night.

**How to read it.**

- **The centre** (small u, v) holds short baselines: large structures on the sky.
- **The edges** hold long baselines: small details. The largest distance from the centre sets the finest detail we can see (the **resolution**).
- **Gaps** in the coverage produce **sidelobes** (patterns of false ripples) around sources in the image.

**What we see:** a very full, round coverage, densest in the middle. This is MeerKAT's dense core, plus 11 hours of Earth rotation. Full, round coverage gives a clean, round beam, which is ideal for HI imaging. The longest baselines reach about 32,000 wavelengths, which corresponds to a resolution of a few arcseconds. In practice we weight the short baselines more (step 13), so our HI images have a resolution of about 25–30 arcseconds.

## What can go wrong

{: .problem }
> **A dead or broken antenna.** Its amplitude is near zero, or very noisy, or jumps.
>
> **How to notice:** one panel in the per-antenna plots looks different from all the others.
>
> **Cure:** flag that antenna (step 3). If only part of the time is bad, flag only that time range.

{: .problem }
> **Strong RFI.** Spikes or broad bumps in amplitude against frequency, often in the same channels for all antennas. MeerKAT L band has known RFI from GPS (around 1176, 1227 and 1381 MHz), Galileo and GLONASS satellites, aircraft (around 1030 and 1090 MHz) and mobile phones (around 900 MHz).
>
> **How to notice:** amplitude against frequency (or channel) plots.
>
> **Cure:** flag those channels, by hand (`mode='manual'`) or with automatic flaggers such as `tfcrop` and `rflag` in CASA, or AOFlagger.

{: .problem }
> **A plot fails with "DISPLAY environment variable is not set".** plotms in CASA 6 needs a graphics display, even when it only writes a PNG file. Compute nodes on a cluster have no screen.
>
> **Cure:** run a "virtual screen" (Xvfb) during the job. Our slurm script starts Xvfb from a container, because the cluster did not have it installed.

{: .problem }
> **Polarisation mixed up in a plot.** On calibration tables (steps 5–8), CASA calls the polarisations **X** and **Y**, not XX and YY. Asking for `correlation='XX'` on a calibration table makes the plot fail.
