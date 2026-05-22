[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on005262-blue)](https://doi.org/10.82901/nemar.on005262)

# ArEEG: Arabic EEG Dataset

This dataset is a collection of Inner Speech EEG recordings from 12 subjects, 7 males and 5 females with visual cues written in Modern Standard Arabic.
Go to [GitHub Repository](https://github.com/Eslam21/ArEEG-an-Open-Access-Arabic-Inner-Speech-EEG-Dataset) for usage instructions.

## NEMAR curation changes (2026-05-21)

BIDS validator: 2790 errors + 15626 warnings -> 0 errors + 10045 warnings. Raw BrainVision binary payloads (`.eeg`, `.vhdr`, `.vmrk`) unchanged.

### `task-innerspeech_eeg.json` (new, root inheriting sidecar)
- Added a single root sidecar carrying the values that are invariant across all 186 per-session recordings. BIDS inheritance broadcasts it to every `sub-*/ses-*/eeg/sub-*_ses-*_task-innerspeech_eeg.{eeg,vhdr,vmrk}` triplet. Why: the dataset shipped with no `_eeg.json` sidecars at all, so every required-EEG-key validator check (`TaskName`, `SamplingFrequency`, `PowerLineFrequency`, `EEGReference`, `SoftwareFilters`) fired once per file extension per recording = 558 x 5 = 2790 `SIDECAR_KEY_REQUIRED` errors. One root sidecar with these five keys closes all of them.
- `TaskName: "innerspeech"`. Why: matches the `task-innerspeech` entity in every recording filename.
- `SamplingFrequency: 250`. Why: derived from the BrainVision header (`SamplingInterval=4000` microseconds = 250 Hz) and confirmed against the harness `raw_meta.json` (`loaded.sfreq=250.0` for all 186 recordings).
- `PowerLineFrequency: "n/a"`. Why: BIDS allows `"n/a"` when the value is not documented; the recording site's grid frequency is not stated in the source dataset, so inferring 50 vs 60 Hz from author affiliations would violate the "do not invent metadata" rule.
- `EEGReference: "Not documented in source recording"`. Why: the BrainVision channel-info block declares an empty per-channel reference (`Ch1=Fz,,...`) and the source dataset does not specify the reference scheme anywhere; this string preserves the fact rather than guessing a value.
- `SoftwareFilters: "n/a"`. Why: BIDS-allowed when no software filters were applied; the source dataset documents none.
- `EEGChannelCount: 8`, `EOGChannelCount: 0`, `ECGChannelCount: 0`, `EMGChannelCount: 0`, `MISCChannelCount: 0`, `TriggerChannelCount: 0`. Why: the BrainVision header lists exactly 8 channels (`Fz`, `C3`, `Cz`, `C4`, `Pz`, `PO7`, `OZ`, `PO8`), all standard scalp-EEG electrode labels; no EOG/ECG/EMG/MISC/Trigger labels appear. Closes 5 x 558 = 2790 channel-count `SIDECAR_KEY_RECOMMENDED` warnings.
- `EEGPlacementScheme: "10-10"`. Why: the channel set mixes 10-20 positions (Fz/C3/Cz/C4/Pz) with extended 10-10 positions (PO7/OZ/PO8); per the BIDS convention the broader 10-10 label covers both.
- `RecordingType: "continuous"`. Why: BrainVision binary continuous format (`DataFormat=BINARY`, `DataOrientation=MULTIPLEXED` in the .vhdr header); no epoch boundaries in the data file.
- `TaskDescription`. Why: paraphrased verbatim from this README's existing task description ("Inner Speech EEG recordings ... with visual cues written in Modern Standard Arabic"); closes 558 `SIDECAR_KEY_RECOMMENDED:TaskDescription` warnings.

### `dataset_description.json` (edit)
- Added `GeneratedBy: [{Name: "nemar-cli", Version: "0.8.8", CodeURL: "..."}]`. Why: closes the one `JSON_KEY_RECOMMENDED:GeneratedBy` warning and documents that the dataset passed through the NEMAR curation tooling. `DatasetType: "raw"` was already present, so this addition does not trigger the derivative-rules cascade.

### Out of mechanical scope (warnings deliberately left in place)
- `EVENTS_TSV_MISSING` (558). Why: no per-recording `_events.tsv` files exist; deriving them from the BrainVision `.vmrk` marker files would require per-recording binary inspection (loading 186 .vhdr files via MNE) which sits outside the mechanical-sidecar envelope of this curation pass.
- `SIDECAR_KEY_RECOMMENDED:RecordingDuration` (558). Why: per-recording value (`ntimes / sfreq` varies recording-to-recording), so it cannot go in the root inheriting sidecar; creating 186 per-recording sidecars to close one warning each was judged not worth the file-count cost.
- Tier-C recommended-but-undocumented fields (~558 each, 16 fields): `Manufacturer`, `ManufacturersModelName`, `SoftwareVersions`, `DeviceSerialNumber`, `Instructions`, `CogAtlasID`, `CogPOID`, `InstitutionName`, `InstitutionAddress`, `InstitutionalDepartmentName`, `CapManufacturer`, `CapManufacturersModelName`, `EEGGround`, `HeadCircumference`, `HardwareFilters`, `SubjectArtefactDescription`. Why: none are documented in the source dataset (README, dataset_description.json, BrainVision header, or per-recording sidecars), and filling them with placeholders or guesses would violate the 100% defensibility rule. These need input from the original authors.
- `JSON_KEY_RECOMMENDED:HEDVersion` (1). Why: the dataset does not use HED tags, so declaring an HED schema version would be misleading.
