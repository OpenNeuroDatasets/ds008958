# Sense of Agency in Gaze-Contingent Display Delay fMRI Study

## Overview

This dataset contains whole-brain functional MRI (fMRI), structural T1-weighted MRI, behavioral event logs, and concurrent EyeLink 1000 Plus eye-tracking recordings from **23 healthy adult participants (`sub-01` to `sub-23`)** performing a gaze-contingent visual search and comparison task.

The study investigates the neural correlates of the **sense of agency (SoA)** over oculomotor actions using a gaze-contingent display paradigm in which the temporal delay and spatial congruency between a participant's eye movements and a gaze-contingent viewing window were systematically manipulated.

- **Authors**: Junhui Kim, Takako Yoshida
- **BIDS Version**: `1.9.0`
- **License**: `CC0`
- **Ethics Approval**: Tokyo Institute of Technology Ethics Review Committee for Epidemiological Research (Approval No. `2022386`)

---

## Participants

Twenty-three right-handed healthy volunteers with normal or corrected-to-normal vision participated in the study (`sub-01` to `sub-23`; 19 males, 4 females; age range: 19–25 years). Demographic information and the counterbalanced session order for each participant are provided in `participants.tsv` and `participants.json`.

---

## Experimental Design and Task

### Gaze-Contingent Visual Comparison Task (Go/No-Go)

On each trial (`2.0 s` duration), five Japanese Hiragana characters were arranged in a cross configuration on a masked display (one central cue character and four peripheral target candidates at top-left, top-right, bottom-left, and bottom-right positions).

1. Peripheral characters were occluded by a mask except within a circular **gaze-contingent window** linked to the participant's real-time gaze position measured at `1000 Hz`.
2. Participants visually explored the peripheral characters and performed a **Go/No-Go comparison task**:
   - **Go trial (`target_existence = 1`, 2 trials per block)**: One of the peripheral characters matched the central character; participants pressed the response button as quickly as possible.
   - **No-Go trial (`target_existence = 0`, 3 trials per block)**: None of the peripheral characters matched the central character; participants withheld button presses.

### Experimental Conditions (5 Conditions)

Each functional run included **20 blocks** (4 blocks per condition in pseudo-randomized order):

| Condition Name (`trial_type`) | Condition Code | Temporal Delay (ms) | Spatial Window Control | Description |
| :--- | :---: | :---: | :---: | :--- |
| `0_ms` | `1` | `0` | Congruent (`controllable = 1`) | Real-time gaze-contingent window with no added delay |
| `100_ms` | `2` | `100` | Congruent (`controllable = 1`) | Gaze-contingent window updated with a `100 ms` delay |
| `300_ms` | `3` | `300` | Congruent (`controllable = 1`) | Gaze-contingent window updated with a `300 ms` delay |
| `1000_ms` | `4` | `1000` | Congruent (`controllable = 1`) | Gaze-contingent window updated with a `1000 ms` delay |
| `Incongruency` | `5` | `300` or `1000` | Mirrored (`controllable = -1`) | Gaze-contingent window spatially inverted across the horizontal or vertical axis with a `300 ms` or `1000 ms` delay |

### Block and Run Structure (`600 s` = `300 TRs` per Run)

Each participant completed **4 functional runs** (`run-01` to `run-04`). Every run lasted `600 s` (`300 volumes` at `TR = 2.0 s`) and consisted of **20 blocks** of `30.0 s` (`15 TRs`) each:

1. **Task Period (`16.0 s` = `8 TRs`, onset at $30 \times (b - 1)\text{ s}$)**:
   - Consists of **5 consecutive visual comparison trials** under a single condition.
   - Each trial lasts `2.0 s`, separated by a `1.5 s` inter-trial interval ($5 \times 2.0\text{ s} + 4 \times 1.5\text{ s} = 16.0\text{ s}$).
2. **Authorship Report and Rest Period (`14.0 s` = `7 TRs`, onset at $30 \times (b - 1) + 16.0\text{ s}$)**:
   - `1.0 s` pre-question fixation (`+16.0 s` to `+17.0 s`)
   - `3.0 s` **Subjective Authorship / Agency Judgment (`Q`, `+17.0 s` to `+20.0 s`)**: Participants reported their perceived control over the viewing window during the preceding block using a 3-alternative forced choice:
     - `self` (`2`): *"I felt I was controlling the window myself without delay"*
     - `delay` (`3`): *"I felt I was controlling the window myself, but it was delayed"*
     - `other` (`4`): *"I felt the window was being controlled by someone else / the computer"*
   - `10.0 s` inter-block rest fixation (`+20.0 s` to `+30.0 s`).

---

## MRI Data Acquisition

All MRI data were acquired on a **3T Siemens MAGNETOM Prisma** scanner (`syngo MR E11`) equipped with a 32-channel head coil.

- **Functional BOLD EPI (`sub-XX/func/sub-XX_task-agency_run-01..04_bold.nii.gz`)**:
  - Sequence: T2\*-weighted gradient-echo echo-planar imaging (EPI)
  - Repetition Time (`TR`): `2.0 s` (`2000 ms`)
  - Echo Time (`TE`): `0.03 s` (`30 ms`)
  - Flip Angle: `77°`
  - Field of View (`FOV`): `192 × 192 mm` (`64 × 64` matrix)
  - Voxel Size: `3.0 × 3.0 × 3.0 mm`
  - Slices: `36` axial slices, slice thickness `3.0 mm`, inter-slice gap `0.75 mm` (`3.75 mm` center-to-center spacing), acquired in **descending** order
  - Parallel Imaging: GRAPPA acceleration factor = `2`, Phase Encoding Direction = `j-` (`A >> P`)
  - Volumes per Run: `300` volumes (`600 s`). *(Note: 305 volumes were acquired per run; the first 5 dummy volumes [`10 s`] were discarded prior to analysis to allow for T1 equilibration, so Volume 1 in each 4D NIfTI file corresponds to $t = 0\text{ s}$, the onset of Block 1.)*
- **Structural T1w (`sub-XX/anat/sub-XX_T1w.nii.gz`)**:
  - Sequence: 3D MPRAGE (`t1_mprage_sag_p2`)
  - `TR = 2.3 s`, `TE = 0.00298 s` (`2.98 ms`), `TI = 0.9 s` (`900 ms`), Flip Angle = `9°`
  - Voxel Size: `1.0 × 1.0 × 1.0 mm`, `256 × 256 × 192` sagittal matrix
  - All T1-weighted anatomical images were defaced (`spm_deface` in SPM12) prior to sharing to protect participant privacy.

---

## Dataset Structure

```text
├── dataset_description.json
├── participants.tsv
├── participants.json
├── README.md
├── CHANGES
├── T1w.json
├── task-agency_bold.json
├── task-agency_events.json
├── sub-01 .. sub-23/
│   ├── anat/
│   │   ├── sub-XX_T1w.nii.gz
│   │   └── sub-XX_T1w.json
│   └── func/
│       ├── sub-XX_task-agency_run-01..04_bold.nii.gz
│       ├── sub-XX_task-agency_run-01..04_bold.json
│       └── sub-XX_task-agency_run-01..04_events.tsv
├── sourcedata/
│   ├── eyetracking/
│   │   └── sub-01 .. sub-23/
│   │       ├── sub-XX_task-agency_run-01..04_eyetrack.edf
│   │       └── sub-XX_task-agency_acq-practice_eyetrack.edf
│   └── behavior_raw/
│       ├── fMRI_main_csv/
│       ├── fMRI_practice_csv/
│       └── preliminary_darkroom_csv/
└── code/
    ├── stimulus_presentation/   # C/SDL stimulus presentation source code (main.c, trial.c, gaus.c, gcwindow.h)
    ├── documentation/           # Experimental methods & contrast specifications
    └── conversion/              # BIDS conversion script
```

### Event File Format (`sub-XX_task-agency_run-01..04_events.tsv`)

Each `*_events.tsv` file contains 140 rows per run (ordered chronologically by `onset`):
- **20 Block-level rows** (`event_level = block`, `onset = 0, 30, ..., 570 s`, `duration = 16.0 s`, `trial_type` $\in$ `{0_ms, 100_ms, 300_ms, 1000_ms, Incongruency}`): Used directly for block-design fMRI GLM modeling.
- **20 Authorship Judgment rows** (`event_level = question`, `onset = 16, 46, ..., 586 s`, `duration = 4.0 s`, `trial_type = Q`): Represents the post-block authorship rating period (`agency_judgment_code` $\in$ `{2, 3, 4}` and `agency_judgment_label` $\in$ `{self, delay, other}`).
- **100 Trial-level rows** (`event_level = trial`, `duration = 2.0 s`, `trial_type` $\in$ `{trial_0_ms, trial_100_ms, trial_300_ms, trial_1000_ms, trial_Incongruency}`): Contains trial-by-trial Go/No-Go target presence, button response, reaction time, gaze arrival time, and saccade target direction. Full column definitions are provided in `task-agency_events.json`.
