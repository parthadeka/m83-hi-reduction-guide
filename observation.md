---
title: "0. The observation"
nav_order: 2
permalink: /observation/
---

# 0. The observation
{: .no_toc }

Before we touch the data, we need to know what is inside it.
{: .fs-5 .fw-300 }

1. TOC
{:toc}

---

## How a radio interferometer works (very short version)

MeerKAT has 64 dishes, each 13.5 m wide. In this observation **60 dishes** (we call them **antennas**) took part. Their names are `m000` to `m063`; four numbers are missing because those antennas were not used.

A single dish only sees a blurry picture of the sky. An **interferometer** combines the signals from *pairs* of antennas. One pair is called a **baseline**. With 60 antennas there are 60 × 59 / 2 = **1770 baselines**.

For each baseline, a machine called the **correlator** multiplies the signals of the two antennas together and averages them for a short time. The result is one complex number, called a **visibility**. A complex number has two parts. We usually write it as:

- an **amplitude** (how strong the signal is), and
- a **phase** (an angle, in degrees, which says how the two waves line up in time).

Every 8 seconds (this is the **integration time**, or **dump time**), the correlator writes one visibility for every baseline, every frequency channel and every polarisation. Later, a computer turns millions of visibilities into an image. This is called **imaging**.

{: .plain }
> Long baselines (antennas far apart) see small details on the sky. Short baselines (antennas close together) see large, smooth structures. We need both to make a good image.

## What was observed

| Item | Value |
|:-----|:------|
| Telescope | MeerKAT, South Africa |
| Observation ID | 1575768670 |
| Date | 8 December 2019, 01:31 to 12:30 UTC (about 11 hours) |
| Antennas | 60 |
| Frequency range recorded | 856 to 1712 MHz (the whole "L band") |
| Number of channels | 32768, each 26.1 kHz wide ("32k mode") |
| Polarisations | XX, XY, YX, YY |
| Time between data points | 8 seconds |

### Frequency channels

The correlator cuts the received frequency range into many narrow slices, called **channels**. Here each channel is **26.1 kHz** wide. Near 1420 MHz, 26.1 kHz is the same as a velocity step of about **5.5 km/s** (because of the Doppler effect, see below). This fine slicing is what lets us measure how the gas moves.

### Polarisations

Each MeerKAT antenna has two **feeds** (receivers). One picks up radio waves that vibrate horizontally (**X**), the other vertically (**Y**). When the correlator combines antenna 1 with antenna 2, it makes four products:

- **XX** and **YY** ("parallel hands"): X with X, and Y with Y. These carry almost all of the signal from unpolarised sources such as HI.
- **XY** and **YX** ("cross hands"): these are only needed for polarisation studies.

HI emission is not polarised, so we keep only **XX and YY**. Total brightness (called **Stokes I**) is the average of XX and YY.

### The fields

The telescope looked at 12 different positions on the sky. Each position is called a **field**.

| Field | What it is | Why we observed it |
|:------|:-----------|:-------------------|
| **J1939-6342** | A very bright, well-known radio source (about 15 Jy) | **Flux calibrator** (sets the brightness scale), **bandpass calibrator** and **delay calibrator** |
| **PKS1313-333** | A bright compact source close to M83 on the sky | **Gain calibrator** (also called **phase calibrator**). Observed every ~15 minutes to follow changes with time |
| J1311-2216 | Another compact source near M83 | A second (spare) gain calibrator. Not used for the final calibration |
| J0408-6545 | Another bright standard source | A possible flux/bandpass calibrator. Not used in this reduction |
| J1331+3030 | 3C286, a standard polarisation calibrator | Only needed for polarisation work. Not used |
| **M83_0 … M83_6** | Seven pointings around M83 | The **target**: this is the science |

The unit **Jy** (jansky) measures how bright a radio source is. 1 Jy = 10<sup>−26</sup> W m<sup>−2</sup> Hz<sup>−1</sup>. Radio sources are weak, so we often use mJy (millijansky, 1/1000 Jy) and µJy (microjansky, 1/1,000,000 Jy).

### A mosaic of seven pointings

An antenna can only see a limited patch of sky at one time. This patch is called the **primary beam**. For MeerKAT at 1.42 GHz it is about **1 degree** wide (we measure the width where the sensitivity drops to half: the **FWHM**, "full width at half maximum").

The HI disk of M83 is bigger than 1 degree. So the observers pointed the telescope at **seven positions in a hexagon**, about 42 arcminutes apart, and visited them one after another. Later, we combine the seven into one large image. This is called a **mosaic**.

```
            M83_2                M83_5

   M83_1             M83_4              M83_6
                  (centre, on M83)

            M83_0                M83_3
```

### The order of the observation

The schedule repeats a simple pattern, many times over 11 hours:

1. gain calibrator PKS1313-333 (2 minutes),
2. the seven M83 pointings (about 1.5 minutes each),
3. the spare gain calibrator J1311-2216 (1 minute),
4. back to PKS1313-333 …

Three times during the night (around 05:50, 08:20 and 10:45 UTC), the telescope also went to J1939-6342 for about 10 minutes.

Each M83 pointing collected about **54 minutes** of data in total. Each visit to one field is called a **scan**. There are 315 scans in the whole observation.

{: .plain }
> The gain calibrator is visited again and again because the atmosphere and the electronics change slowly with time. By looking at a source we understand well, every 15 minutes, we can measure those changes and remove them from the target data.

## What the final result should look like

At the end, we want a **data cube**: a stack of images, one for each frequency channel. Because of the **Doppler effect**, gas moving towards us appears at a slightly higher frequency, and gas moving away appears at a lower frequency. So each channel of the cube shows the gas that moves at one particular speed. M83 rotates, so different parts of the disk appear in different channels. M83's gas has velocities from about **360 to 670 km/s** (M83 as a whole moves away from us at about 513 km/s).
