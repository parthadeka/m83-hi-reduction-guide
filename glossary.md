---
title: Glossary
nav_order: 4
permalink: /glossary/
---

# Glossary
{: .no_toc }

All the technical words used in this guide, in alphabetical order, explained simply.
{: .fs-5 .fw-300 }

---

**Absorption (HI absorption)**
: Cold HI gas in front of a bright continuum source absorbs a little of that source's light at the HI frequency. It appears as a dip in the spectrum.

**Amplitude**
: The strength of a visibility (or of a gain). One of the two parts of a complex number; the other is the phase.

**Antenna**
: One dish of the interferometer. MeerKAT antennas are 13.5 m wide and are called m000 to m063.

**Autocorrelation**
: An antenna combined with itself. It measures total power, not sky structure. We flag it.

**Bandpass**
: How the sensitivity of an antenna changes with frequency, channel by channel.

**Baseline**
: A pair of antennas. With 60 antennas there are 1770 baselines. Short baselines see large structures; long baselines see small details.

**Beam**
: Used for two different things. The **primary beam** is the sensitivity pattern of one dish on the sky (about 1° wide for MeerKAT at 1.4 GHz). The **synthesised beam** (or PSF) is the image of a point source made by the whole interferometer (about 25–30″ in our HI cube).

**Beam smearing**
: The widening of spectra (and flattening of velocity gradients) because one beam covers gas at different velocities. Strongest near a galaxy centre, where rotation changes fast with position.

**Briggs weighting (robust)**
: A weighting scheme between natural (`robust=+2`) and uniform (`robust=−2`). See *weighting*.

**Calibration**
: Measuring and removing the effects of the telescope and atmosphere on the data.

**Calibrator**
: A source with known properties, used to measure the telescope's errors. See *flux calibrator*, *bandpass calibrator*, *gain calibrator*.

**CARTA**
: Cube Analysis and Rendering Tool for Astronomy. An interactive viewer for large images and cubes.

**CASA**
: Common Astronomy Software Applications. The standard software for reducing radio interferometer data.

**Channel**
: One narrow slice of frequency. Ours are 26.1 kHz wide (about 5.5 km/s at 1420 MHz).

**Clean / CLEAN / deconvolution**
: The process that removes sidelobes from a dirty image, by building a model of the sky out of clean components.

**Clean component**
: A small piece of flux, at one position, added to the model during cleaning.

**Clipping**
: Keeping only values above a limit (for example 3σ).

**Column density**
: The number of atoms in a column through the gas, per unit area. Proportional to the moment 0 map for HI.

**Complex number**
: A number with two parts, written as amplitude and phase. Visibilities and gains are complex numbers.

**Continuum emission**
: Radio emission spread over all frequencies, changing slowly with frequency. Compare *line emission*.

**Continuum subtraction**
: Removing the continuum from the data, so that only line emission remains.

**Correlator**
: The machine that combines the signals of every pair of antennas and produces the visibilities.

**Cube (data cube)**
: A 3D image: two sky axes and one frequency (or velocity) axis.

**Decorrelation**
: Loss of signal when complex numbers with different phases are averaged, so that they partly cancel.

**Delay**
: A timing difference between the signals of two antennas. It makes the phase change linearly with frequency.

**Direction-dependent errors / calibration**
: Calibration errors that are different in different directions of the field (from the primary beam shape, dish pointing or the ionosphere). They show as rings or "spokes" around bright sources far from the pointing centre. Direction-dependent calibration (for example with killMS and DDFacet) solves gains for several directions separately.

**Dirty image / dirty beam**
: The image made directly from the gridded visibilities, before cleaning; and the image of a point source made the same way.

**Doppler effect**
: Moving sources appear at a shifted frequency: lower if moving away, higher if moving towards us. This lets us measure gas velocities from the HI line.

**Dump time / integration time**
: How often the correlator writes out data. Here every 8 seconds.

**Dynamic range**
: The brightest value in an image divided by the noise. It measures how well bright and faint things can be seen in the same image.

**Field**
: One position on the sky that the telescope looked at.

**FITS**
: Flexible Image Transport System, the standard file format for astronomical images.

**Flag / flagging**
: Marking data as bad, so that it is not used. Flags can be undone.

**Flux calibrator**
: A source with accurately known brightness, used to set the flux scale. Ours is J1939-6342.

**Flux density**
: The brightness of a source, measured in janskys (Jy).

**Flux scale transfer (fluxscale)**
: Using the flux calibrator to find the true brightness of the gain calibrator.

**Fourier transform**
: The mathematical operation that links the visibilities (uv plane) and the image (sky plane).

**FWHM**
: Full width at half maximum: the width of a peak measured where it drops to half its height.

**Gain**
: How much an antenna changes the signal: its amplitude (amplification) and phase (time shift). Gains change with time and frequency.

**Gain calibrator (phase calibrator)**
: A bright, compact source near the target, observed regularly, to follow how the gains change with time. Ours is PKS1313-333.

**Global profile**
: The total spectrum of a galaxy: the flux summed over the whole galaxy, in each channel.

**Gridding**
: Putting the visibilities on a regular grid before the Fourier transform.

**HI**
: Neutral atomic hydrogen. It emits a spectral line at 1420.405752 MHz (21 cm wavelength).

**I/O bound**
: A job whose speed is limited by reading and writing the disk, not by the CPU.

**Interferometer**
: A telescope made of many antennas whose signals are combined in pairs.

**Interpolation**
: Estimating a value between two measurements. We interpolate the gains linearly between calibrator scans.

**Jansky (Jy)**
: The unit of flux density in radio astronomy. 1 Jy = 10<sup>−26</sup> W m<sup>−2</sup> Hz<sup>−1</sup>. 1 mJy = 0.001 Jy; 1 µJy = 0.000001 Jy.

**katbeam**
: SARAO's Python model of the measured MeerKAT primary beam (for L band: `JimBeam('MKAT-AA-L-JIM-2020')`).

**Kinematic major axis**
: The direction across a rotating disk along which the measured velocities change most: where the gas moves most directly towards and away from us.

**Line emission**
: Emission at one specific frequency, such as the HI line.

**Line-free channels**
: Channels without any line emission. Used to measure and remove the continuum, and to measure noise.

**LSRK**
: Kinematic local standard of rest. A reference frame for velocities that removes the motion of the Earth and Sun.

**MAD (median absolute deviation)**
: A robust way to measure noise: the median of the distances from the median. Multiplied by 1.4826 it equals the RMS for pure noise, but a few bright sources hardly change it.

**Major / minor cycle**
: In cleaning: a major cycle goes through all the visibilities to make a new residual image; minor cycles find clean components in the image between major cycles.

**Mask**
: A region (2D) or set of voxels (3D) that marks where real emission is. Cleaning and moment maps can be limited to the mask.

**Measurement Set (MS)**
: CASA's data format for visibilities: a folder with many tables inside.

**Model (MODEL_DATA)**
: What we expect the data to look like. For calibrators, set by `setjy`. In the continuum-subtracted MS, it holds the fitted continuum.

**Moment maps (0, 1, 2)**
: 2D maps made from a cube: the total amount of gas (0), the average velocity (1), and the spread of velocities (2).

**Mosaic**
: An image made by combining several pointings of the telescope.

**mpicasa**
: The tool that runs CASA on several CPU cores at once (in parallel).

**Multi-scale clean**
: A clean that uses components of several sizes (points and smooth blobs), instead of points only. Much better for extended emission.

**Natural weighting**
: All visibilities get the same weight. Lowest noise, larger beam.

**Noise (RMS, σ)**
: The random fluctuations in the data or image. Usually measured as the RMS (root mean square). "3σ" means three times the noise.

**Optical depth (τ)**
: How strongly gas absorbs: the fraction of light that gets through is e<sup>−τ</sup>. For HI absorption, τ = −ln(S/S<sub>c</sub>).

**Parallactic angle**
: The angle between the dish (its feeds and beam) and the sky. It changes as a source moves across the sky, so a fixed, non-round beam turns on the sky during an observation.

**Phase**
: The angle part of a complex number. For a visibility it says how the waves from the two antennas line up in time. Only phase *differences* and *changes* matter.

**Phase centre**
: The sky position at the centre of a field. Visibilities are measured relative to it.

**Phase wrapping**
: Phase is an angle, so +180° and −180° are the same. A phase that drifts past 180° jumps to −180° in a plot.

**Point source**
: A source much smaller than the resolution. Its visibilities have the same amplitude on every baseline.

**Polarisation (XX, YY, XY, YX)**
: Each antenna has two receivers, X and Y. XX and YY are the products of the same receivers on the two antennas; XY and YX mix them.

**Primary beam correction (pbcor)**
: Dividing an image by the primary beam response, so brightness is correct everywhere. The noise then rises towards the edges.

**PSF (point spread function)**
: The synthesised (dirty) beam: the image of a point source.

**Radiometer equation**
: The formula that predicts the noise from the system temperature, dish area, bandwidth, time and number of baselines.

**Reference antenna (refant)**
: The antenna whose phase is set to zero. All other phases are measured relative to it.

**Reference frame (TOPO, LSRK, BARY)**
: The frame in which frequencies and velocities are measured. TOPO moves with the telescope; LSRK and BARY remove the Earth's motion.

**Reliability filter**
: SoFiA's test that removes detections likely to be noise, by comparing the positive detections with the negative ones that pure noise also produces.

**Residual**
: What is left in the image after the clean components have been removed.

**Resolution**
: The smallest detail that can be seen, set by the beam size.

**Restoring beam**
: The clean Gaussian beam used to smooth the clean components into the final image.

**RFI**
: Radio frequency interference: man-made radio signals (satellites, aircraft, phones) that spoil the data.

**Scan**
: One continuous visit of the telescope to one field.

**Self-calibration (selfcal)**
: Using a model of the target field itself to improve the gain calibration.

**Shadowing**
: When one dish blocks the view of another, at low elevation. Shadowed data are flagged.

**Sidelobes**
: The rings and ripples around a source in a dirty image, caused by gaps in the uv coverage.

**Signal-to-noise ratio (SNR)**
: How strong a signal is compared with the noise.

**slurm**
: The system that queues and runs jobs on the computing cluster.

**SoFiA**
: Source Finding Application. Software that finds HI sources in a cube and makes masks, moment maps and catalogues.

**Solution interval (solint)**
: How much data (in time or channels) goes into one calibration solution.

**Spectral index**
: How the brightness of a source changes with frequency.

**Spectral window (spw)**
: A block of frequency channels. Our data has one, number 0.

**Spin temperature (T<sub>s</sub>)**
: A temperature that describes how the HI atoms are spread over the two levels of the 21 cm line. For cold gas it is close to the gas temperature. The HI column density from absorption scales with it.

**Standing wave**
: A ripple in the bandpass caused by signal reflections inside the receiver or between the dish and the receiver.

**Stokes I**
: Total intensity: the average of XX and YY.

**System temperature (T<sub>sys</sub>)**
: A measure of how much noise the receiver and the sky add to the signal.

**Systemic velocity**
: The velocity of a galaxy as a whole. M83's is about 513 km/s.

**tclean**
: CASA's task for imaging and cleaning.

**Threshold**
: The level at which cleaning stops.

**uv coverage / uv plane**
: The set of all baselines, as seen from the source, in wavelengths. More complete coverage gives cleaner images.

**uv range (kλ)**
: A selection of baselines by their length, in thousands of wavelengths (kλ). Long baselines only see small structures; at 21 cm, 5 kλ is about 1 km.

**Visibility**
: The basic measurement of an interferometer: one complex number per baseline, time, channel and polarisation.

**Voxel**
: One element of a cube: one sky pixel in one channel.

**W20, W50**
: The width of an HI profile, measured at 20% and 50% of its peak.

**Warp**
: A disk whose outer parts are tilted differently from its inner parts. It shows as twisted lines of equal velocity in a velocity field.

**Weighting**
: How much each visibility counts when making the image. Natural, uniform and Briggs (robust) weighting give different beams and noise.

**WSClean**
: A fast imaging and cleaning program, widely used for MeerKAT data.
