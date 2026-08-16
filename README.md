[![DOI](https://img.shields.io/badge/DOI-10.82901%2Fnemar.on005262-blue)](https://doi.org/10.82901/nemar.on005262)

# ArEEG: Arabic EEG Dataset

This dataset is a collection of Inner Speech EEG recordings from 12 subjects, 7 males and 5 females with visual cues written in Modern Standard Arabic.
Go to [GitHub Repository](https://github.com/Eslam21/ArEEG-an-Open-Access-Arabic-Inner-Speech-EEG-Dataset) for usage instructions.

## NEMAR curation changes (2026-05-21, revised 2026-05-27)

The BIDS validator went from 2790 errors + 15626 warnings to 0 errors + 10046 warnings. None of the raw `.eeg` files were modified — every change is to a text sidecar.

**Root recording sidecar added (`task-innerspeech_eeg.json`)**

The dataset shipped with no `_eeg.json` sidecars at all, which is what produced the 2790 validator errors (the required EEG keys were missing for every one of the 186 recordings, multiplied across file extensions). A single sidecar was placed at the dataset root; BIDS inheritance applies it to every `sub-*/ses-*/eeg/sub-*_ses-*_task-innerspeech_eeg.{eeg,vhdr,vmrk}` triplet. The fields it carries:
- `TaskName: "innerspeech"` — matches the `task-innerspeech` entity in every recording filename.
- `SamplingFrequency: 250` — read from the BrainVision header (`SamplingInterval=4000` microseconds works out to 250 Hz) and confirmed against the loaded data for all 186 recordings.
- `PowerLineFrequency: "n/a"` — the recording site's grid frequency is not stated anywhere in the source dataset, so the BIDS-allowed `"n/a"` was used rather than guessing 50 or 60 Hz from author affiliations.
- `EEGReference: "Not documented in source recording"` — the BrainVision channel-info block declares an empty per-channel reference and the source dataset doesn't specify the scheme anywhere; this string records the fact rather than inventing a reference electrode.
- `SoftwareFilters: "n/a"` — the source dataset documents no software filters, and `"n/a"` is BIDS-allowed when none were applied.
- `EEGChannelCount: 8`, with `EOGChannelCount`, `ECGChannelCount`, `EMGChannelCount`, `MISCChannelCount`, and `TriggerChannelCount` all set to `0`. The BrainVision header lists exactly 8 channels (`Fz`, `C3`, `Cz`, `C4`, `Pz`, `PO7`, `OZ`, `PO8`), all standard scalp-EEG labels; no auxiliary or trigger channels appear.
- `EEGPlacementScheme: "10-10"` — the channel set mixes 10-20 positions (Fz/C3/Cz/C4/Pz) with extended 10-10 positions (PO7/OZ/PO8); the broader 10-10 label covers both per BIDS convention.
- `RecordingType: "continuous"` — the BrainVision file is a continuous binary recording (`DataFormat=BINARY`, `DataOrientation=MULTIPLEXED`), with no epoch boundaries.
- `TaskDescription` — paraphrased from this README's existing task description ("Inner Speech EEG recordings ... with visual cues written in Modern Standard Arabic") so the validator has a recommended description to read.

**Dataset description (`dataset_description.json`)**
- Updated BIDSVersion from 1.9.0 to 1.11.1 (the version the current validator checks against).
- GeneratedBy was left absent, exactly as the source published it — nothing was added there.

**Remaining warnings (10046) — left on purpose**

This dataset has a lot of recommended-but-missing fields that need information from the study, lab, or equipment that isn't in the dataset, plus a few structural items that sit outside a mechanical sidecar pass. They were left blank rather than filled with guesses:
- Per-recording event tables (`_events.tsv`) are not present. Deriving them from the BrainVision `.vmrk` marker files would require loading every recording in MNE, which goes beyond a text-sidecar cleanup.
- Per-recording duration varies from file to file, so it can't live in the root sidecar; creating 186 per-recording sidecars just to record a duration was judged not worth the file-count cost.
- Hardware and lab descriptors that aren't documented anywhere in the source — manufacturer and model name, software versions, device serial number, cap manufacturer and model, head circumference, hardware filters, ground electrode, institution name and address and department, instructions, cognitive-atlas IDs, and subject-artefact notes. None of these can be filled in without contacting the original authors.
- The HED schema version, since this dataset doesn't use HED tags.
- GeneratedBy on the dataset description, intentionally left absent so the file matches what the source published.
