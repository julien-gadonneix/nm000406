[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.nm000406-blue)](https://doi.org/10.82901/nemar.nm000406)

# Microelectrode recordings (MER) from deep brain stimulation surgery in Parkinson's disease

Intraoperative microelectrode recordings (MER) from **14 patients with Parkinson's disease** (9 male, 5 female;
65.1 ± 5.6 years) during stereotactic surgery to implant deep brain stimulation (DBS) electrodes in the subthalamic
nucleus (STN). Re-packaged in iEEG-BIDS from the authors' public release (Zenodo record
[7352058](https://doi.org/10.5281/zenodo.7352058), "Microelectrode register (MER) data from Deep Brain Stimulation
(DBS) surgery in Parkinson's disease patients", Rosado-Muñoz, Guerrero-Martínez and Gutierrez Martín, CC-BY-4.0).

The release description says: "MER data consist of brain signal in different depths when DBS surgery is being done.
In each depth, a data file is created, with different duration depending on the depth, and up to three channels."
Depths are grouped into STN-IN and STN-OUT (inside / outside the subthalamic nucleus), "classification [...] made by
the neurophysiologists and surgeons". Eight patients have left and right trajectories, one right only and five left
only. "No filtering or data processing was made, they are directly obtained from the MER acquisition system."

The record lists one related article as using these data: Tepper et al. (2017), *Selection of the Optimal Algorithm
for Real-Time Estimation of Beta Band Power during DBS Surgeries in Patients with Parkinson's Disease*, Computational
Intelligence and Neuroscience 2017:1512504 (https://doi.org/10.1155/2017/1512504). It describes MER from 9 bilateral
STN-DBS surgeries at La Fe Hospital, Valencia, Spain, with the MicroGuide system (Alpha Omega) and tungsten
microelectrodes, amplified 10000 times, band-pass 200-6000 Hz, 12 kHz sampling, 12-bit A/D. The release does not say
which of its 14 patients are those 9, so these settings are not copied into the sidecars.

## Layout
- `sub-<NN>` = release patient `P<NN>` (P1-P14). Age and sex are given only at group level, so `participants.tsv` has
  `n/a` for them.
- `task-mer`, `acq-left` / `acq-right` = trajectory side, `run-<k>` = one release file (one depth). Runs are numbered per
  side in ascending order of the release file name (the "depth label"). The release does not document this label's
  unit or origin; it is kept verbatim in `sub-<NN>_scans.tsv` (`source_depth_label`) together with the STN-IN / STN-OUT
  classification (`stn_classification`) and the source path.
- Three channels per file (`Electrode1`-`Electrode3`, as named in the release; type SEEG, units n/a: the release stores
  int16 values without a scale factor). Channel 1 is close to flat in most files; the release does not say why.
- Sampling frequency per run is computed from the release time vector as (N-1)/time[-1]: 12000 Hz in 432 files and
  about 12019 Hz in the other 398 (exact value per run in the sidecar and in `scans.tsv`).
- In 226 of the 830 files the three channels have different lengths (differences are whole 752-sample blocks). The
  release does not document how channels of unequal length line up. Every channel is written from its first sample, and
  shorter channels are NaN after their last sample. `channels.tsv` gives each channel's source length
  (`source_n_samples`), and the sidecar's `RecordingDescription` repeats the lengths.
- There are no electrode coordinates in the release. `electrodes.tsv` (per side) lists the three channels with `n/a`
  coordinates.

## What was converted, and how
Each `.mat` (struct `signal`: `channel`, `data` int16, `time`) became one BrainVision run (IEEE float32: int16 values
are exact). The round-trip check compares every run with its source file. No filtering, resampling or re-referencing
was applied. `sourcedata/zenodo-7352058/data/...` holds the original `.mat` files. The only change is that the MATLAB
header text "Created on" (file export date, 2018 or 2021) has its weekday and day masked (`--- Mon 01`). Year, month
and all data are unchanged, and the provenance JSON gives the SHA-256 of each file before and after masking.

## License

CC-BY-4.0

## Ethics

The recordings in this dataset are covered by the Ethical Committee for Biomedical Research of La Fe Hospital (Valencia, Spain), registration number 2015/0824, approved 17 May 2016. Patients signed an informed consent for DBS surgery with microelectrode recording (MER).

Statement from Bellino et al. (2019), section 2.1 "Database" (verbatim), describing MER from DBS surgery at La Fe Hospital by the same group (J. Guerrero, A. Rosado-Muñoz):

> All of the patients met medical accepted selection criteria and signed an informed consent for DBS surgery with MER. These investigations that use human data were carried out following the rules of the Declaration of Helsinki. The Ethical Committee for Biomedical Research of La Fe Hospital approved the research procedures for the project with registration number 2015/0824 in May 17, 2016.

Bellino GM, Schiaffino L, Battisti M, Guerrero J, Rosado-Muñoz A (2019). Optimization of the KNN Supervised Classification Algorithm as a Support Tool for the Implantation of Deep Brain Stimulators in Patients with Parkinson's Disease. Entropy 21(4):346. https://doi.org/10.3390/e21040346 (PMC7514830).

The Zenodo record 7352058 itself contains no ethics statement. Coverage of the recordings in this release by La Fe approval 2015/0824 is attested by the depositor (Bruno Aristimunha, 2026-10-08).

## NaN samples in the signal files (checked 18 Oct 2026)

Some signal files store NaN (not-a-number) samples. A scan of 590 of the 830 recordings (whole files read with MNE on 18 Oct 2026; the 240 largest were not scanned) found them in 166 recordings. Both MNE-BIDS and EEGDash load these files; `get_data()` returns the NaN values.

In 166 recordings channels are NaN only in part of the recording (166 spans at the end; median 0.13% of the samples; channels most often affected: Electrode2 (114), Electrode3 (87), Electrode1 (80)). Examples: `sub-01_task-mer_acq-left_run-01_ieeg.vhdr` Electrode2, Electrode3 samples 493312-494816; `sub-01_task-mer_acq-left_run-02_ieeg.vhdr` Electrode1, Electrode3 samples 502336-503840; `sub-01_task-mer_acq-left_run-03_ieeg.vhdr` Electrode2, Electrode3 samples 512112-512864; `sub-01_task-mer_acq-left_run-04_ieeg.vhdr` Electrode1 samples 488048-489552; `sub-01_task-mer_acq-left_run-08_ieeg.vhdr` Electrode3 samples 512864-513616. These samples cannot be restored from the release; crop them or replace them (e.g. `numpy.nan_to_num`, interpolation) before filtering. The data files are unchanged.

Per-recording details (channels, NaN sample spans): `code/fix-20261018_nan_scan.json`.
