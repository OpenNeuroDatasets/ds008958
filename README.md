# BIDS Conversion Specification & Staging Directory Guide

This directory (`BIDS_staging/`) contains the complete, curated source files for **all 23 participants (`sub-01` to `sub-23`)** required to construct a BIDS-compliant dataset (including Raw MRI, Behavioral events, EyeLink eye-tracking, SPM12 preprocessing/1st-level/2nd-level derivatives, and Leave-One-Subject-Out [LOSO] derivatives).

Redundant intermediate 3D volume series (`r*`, `ar*`, `war*`, ~78 GB) and duplicate initial conversion folders (`Nii_Converted/`, ~12.2 GB) have been excluded to keep this staging folder clean and self-contained.

---

## 1. Directory Layout of `BIDS_staging/`

```text
BIDS_staging/
├── README_FOR_CLAUDE_AGENT.md              # This specification document
├── info/                                   # Reference documentation & metadata tables
│   ├── participants_mapping.tsv            # Master 23-subject mapping table (IDs, initials, sequences, notes)
│   ├── Method.pdf                          # Manuscript Materials & Methods section
│   ├── Kiroku_original.xlsx                # Original experiment logbook (schedules, initials, notes)
│   ├── Kiroku_participants.xlsx            # Cleaned participant metadata table
│   ├── SPM12_Conditions_per_session.xlsx   # Condition sequences (1..5), block onsets, and trial T1I1 arrays
│   ├── Nii_Batching_README_contrasts.docx  # List of all 22 1st-level GLM contrasts
│   ├── LOSO_What_I_did_here.docx           # LOSO (rSMG) methodology notes
│   ├── LOSO_AG_What_I_did_here.docx        # LOSO (Angular Gyrus) methodology notes
│   └── Eye_movements_spec.docx             # EyeLink EDF message and event parsing specification
├── raw_nii/                                # Raw NIfTI files for all 23 subjects (sub_01 .. sub_23)
│   └── sub_XX/
│       ├── BOLD_DESCEND_1..4/              # 300 3D NIfTI volumes (*_0006.nii to *_0305.nii) + *_info.txt per run
│       └── T1/                             # Raw T1w 3D NIfTI (*_t1_mprage_sag_p2.nii) + *_info.txt
├── behavior_and_eyetracking/               # Behavioral CSVs, EyeLink EDFs, stimulus code & processed tables
│   ├── fMRI_main/                          # 23 subjects × 4 runs: <EyeLinkInitial><1..4> Result Data.csv & .EDF
│   ├── fMRI_practice/                      # In-scanner practice (session 5): <EyeLinkInitial>5 Result Data.csv & .EDF
│   ├── preliminary_darkroom/               # Dark-room preliminary phase: <EyeLinkInitial><1..4> Result Data.csv
│   ├── condition_tempdata/                 # Predefined sequence tables used by SDL program
│   ├── stimulus_code/                      # C source code (main.c, trial.c, gaus.c, gcwindow.h)
│   └── processed_summaries/                # Merged behavioral/GLMM Excel files & Python EDF processing scripts
├── derivatives_spm12/                      # SPM12 (v7771) preprocessing, 1st-level & 2nd-level GLM outputs
│   ├── preproc_and_1st_level_5cond/
│   │   └── sub_XX/
│   │       ├── BOLD_DESCEND_1..4/          # rp_*_0006.txt (6 motion params), mean*_0006.nii, swar*_0006..0305.nii
│   │       ├── T1/                         # w*.nii (MNI-normalized T1w) and y_*.nii (deformation field)
│   │       └── Results/                    # 5-condition 1st-level GLM (SPM.mat, beta_*, con_0001..0022, spmT_*, mask.nii)
│   ├── 1st_level_parametric/
│   │   └── sub_XX/Results_PM/              # Delay-modulated (parametric) 1st-level GLM (SPM.mat, beta_*, con_*, spmT_*)
│   ├── 2nd_level_group/
│   │   ├── Batch_2nd_analysis/             # 3 main group analyses (Factorial ANOVA, Incong vs 0ms, PM One-Sample)
│   │   └── SwE/                            # Repeated-Measures Sandwich Estimator (batch, macro, output)
│   └── spm_batches/                        # SPM12 batch .mat files (Batch_Pre_Process, Batch_1st_analysis, etc.)
└── derivatives_loso/                       # Leave-One-Subject-Out (LOSO) ROI analyses (23 folds each)
    ├── LOSO_rSMG/                          # Right Supramarginal Gyrus LOSO (LOSO_sub_01..23, rSMG ROIs, scripts, xlsx)
    └── LOSO_Angular_Gyrus/                 # Angular Gyrus LOSO (LOSO_sub_01..23, *_roi.mat, scripts, xlsx)
```

---

## 2. Participant ID, Initials, and Session Sequence Mapping (`sub-01` to `sub-23`)

**CRITICAL**: Six participants have **different initials** between the MRI files (`raw_nii/sub_XX/`, `X_Y_...nii`) and the Behavioral/EyeLink files (`behavior_and_eyetracking/fMRI_main/<Initial><1..4>`). Always use `info/participants_mapping.tsv` (also reproduced below):

| BIDS ID | Source Folder | MRI Initial | EyeLink / CSV Initial | Scan Date | Age | Sex | fMRI Run Sequence (Runs 1–4) | Special Notes |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| `sub-01` | `sub_01` | `CY` | `CY` | 2023-03-15 | 25 | F | `3412` | Regular |
| `sub-02` | `sub_02` | `EM` | `EM` | 2023-03-06 | 19 | F | `2341` | Regular |
| `sub-03` | `sub_03` | `GK` | `GK` | 2023-03-07 | 20 | M | `2341` | Run 1 (`GK1`) used session 4 stimulus images with session 1 condition sequence |
| `sub-04` | `sub_04` | **`HA`** | **`HO`** | 2023-03-15 | 20 | M | `3412` | **Initial mismatch** (`HA` vs `HO`) |
| `sub-05` | `sub_05` | **`HH`** | **`HD`** | 2023-03-03 | 19 | M | **`1123`** | **Initial mismatch** (`HH` vs `HD`) + **Irregular sequence `1123`** (scheduled `4123`, but Run 1 ran condition sequence `1`) |
| `sub-06` | `sub_06` | **`JH`** | **`HS`** | 2023-03-06 | 25 | M | `2341` | **Initial mismatch** (`JH` vs `HS`) |
| `sub-07` | `sub_07` | `KI` | `KI` | 2023-03-06 | 19 | M | `1234` | Regular |
| `sub-08` | `sub_08` | **`KM`** | **`KD`** | 2023-03-07 | 19 | F | `1234` | **Initial mismatch** (`KM` vs `KD`) |
| `sub-09` | `sub_09` | `LD` | `LD` | 2023-03-14 | 24 | M | `2341` | Regular |
| `sub-10` | `sub_10` | `LH` | `LH` | 2023-03-14 | 25 | M | `3412` | Regular |
| `sub-11` | `sub_11` | `MG` | `MG` | 2023-03-15 | 19 | M | `3412` | Regular |
| `sub-12` | `sub_12` | `MH` | `MH` | 2023-03-14 | 19 | F | `4123` | Regular |
| `sub-13` | `sub_13` | `MS` | `MS` | 2023-03-08 | 19 | M | `4123` | Debug columns in `MS1`/`MS2` CSV had an index offset bug; actual condition sequence `4123` is intact |
| `sub-14` | `sub_14` | `MT` | `MT` | 2023-03-14 | 19 | M | `4123` | Regular |
| `sub-15` | `sub_15` | `NT` | `NT` | 2023-03-13 | 22 | M | `1234` | Regular |
| `sub-16` | `sub_16` | `OK` | `OK` | 2023-03-13 | 19 | M | `4123` | Regular (Ono Kanta) |
| `sub-17` | `sub_17` | `OT` | `OT` | 2023-03-07 | 25 | M | `1234` | Regular |
| `sub-18` | `sub_18` | `OY` | `OY` | 2023-03-07 | 22 | M | `3412` | Originally recorded as `OK` on 3/7, already renamed to `OY` in `fMRI_main/` |
| `sub-19` | `sub_19` | `SS` | `SS` | 2023-03-06 | 19 | M | `1234` | Regular |
| `sub-20` | `sub_20` | **`UH`** | **`UF`** | 2023-03-13 | 19 | M | `2341` | **Initial mismatch** (`UH` vs `UF`) |
| `sub-21` | `sub_21` | **`WH`** | **`HW`** | 2023-03-08 | 19 | M | `3412` | **Initial mismatch** (`WH` vs `HW`) |
| `sub-22` | `sub_22` | `WI` | `WI` | 2023-03-13 | 19 | M | `1234` | Regular |
| `sub-23` | `sub_23` | `YJ` | `YJ` | 2023-03-08 | 19 | M | `2341` | Regular |

---

## 3. MRI Acquisition Parameters & Volume Numbering

From `info/Method.pdf` (Section 2.7–2.8) and `raw_nii/sub_XX/.../*_info.txt`:

- **Scanner**: 3T SIEMENS Prisma (`syngo MR E11`), 32-channel head coil.
- **Functional BOLD (`func/sub-XX_task-agency_run-01..04_bold.nii.gz`)**:
  - Sequence: T2*-weighted gradient-echo EPI (`epfid2d1_64`, `Usui_BOLD_Descend`)
  - `RepetitionTime` (TR): `2.0` s (2000 ms)
  - `EchoTime` (TE): `0.03` s (30 ms)
  - `FlipAngle`: `77` degrees
  - Voxel size: `3 × 3 × 3` mm, slice thickness `3` mm, slice gap `0.75` mm (center-to-center slice spacing `3.75` mm), FOV `192 × 192 × 134.25` mm (`64 × 64` matrix, 36 axial slices)
  - **Slice Acquisition Order**: Descending (`36` slices acquired in descending order; SPM12 slice-timing correction used TR = 2.0 s, 36 slices, reference slice = 18th slice)
  - `PhaseEncodingDirection`: `j-` / `COL` (`EffectiveEchoSpacing`: `0.00078125` s or `15.63` ms in SPM header note; `PixelBandwidth`: `2440` Hz/px; GRAPPA acceleration factor = `2`)
  - **Volume Count (300 volumes, `_0006.nii` to `_0305.nii`)**:
    - Each functional run originally acquired 305 volumes (610 s).
    - The first 5 dummy volumes (`_0001` to `_0005`, 10 s initial magnetisation stabilisation period) were discarded prior to analysis.
    - Therefore, `BOLD_DESCEND_1..4` contains **300 3D NIfTI files (`_0006.nii` through `_0305.nii`) = 600 seconds**, where time `t = 0 s` (Scan 0 in SPM) corresponds to `_0006.nii` (the onset of Block 1).
    - When converting to BIDS, stack `_0006.nii`..`_0305.nii` in numerical order into a single 4D NIfTI file (`sub-XX_task-agency_run-01..04_bold.nii.gz`).
- **Anatomical T1w (`anat/sub-XX_T1w.nii.gz`)**:
  - Sequence: MPRAGE (`tfl3d1_16ns`, `t1_mprage_sag_p2`)
  - `RepetitionTime` (TR): `2.3` s (2300 ms)
  - `EchoTime` (TE): `0.00298` s (2.98 ms)
  - `InversionTime` (TI): `0.9` s (900 ms)
  - `FlipAngle`: `9` degrees
  - Voxel size: `1 × 1 × 1` mm, FOV `256 × 256 × 192` mm (192 sagittal slices).

---

## 4. Experimental Paradigm, Block/Trial Timing, and `*_events.tsv` Specification

### 4.1 Timeline of Each Run (600 seconds = 300 TRs = 20 Blocks)
Each run consists of **20 blocks**, each lasting exactly **30 seconds (15 TRs)**.
For block $b \in \{1, 2, \dots, 20\}$ (where 0-based index is $b-1$):

1. **Task Period (16 seconds = 8 TRs)**:
   - **Block Onset**: $30 \times (b - 1)$ seconds (i.e., `0, 30, 60, ..., 570` s; in SPM scans: `0, 15, 30, ..., 285` scans).
   - **Block Duration**: `16.0` seconds (`8` scans in SPM).
   - Within this 16 s task period, the participant performs **5 consecutive visual comparison (Go/No-Go) trials** under a single condition:
     - Each trial $t \in \{1, 2, 3, 4, 5\}$ lasts **`2.0` seconds**, separated by a **`1.5` second** inter-trial interval (4 ISIs between the 5 trials: $5 \times 2.0 + 4 \times 1.5 = 16.0\text{ s}$).
     - **Individual Trial Onset**: $30 \times (b - 1) + 3.5 \times (t - 1)$ seconds (`+0.0, +3.5, +7.0, +10.5, +14.0` s relative to block start), duration = `2.0` s.
2. **Rest & Subjective Authorship Report Period (14 seconds = 7 TRs)**:
   - Immediately follows the 16 s task period, spanning $[30(b-1) + 16,\ 30(b-1) + 30)$ seconds:
     - `1.0` s pre-questionnaire fixation (`[+16.0 s, +17.0 s]`)
     - `3.0` s subjective authorship report (`Q`, `[+17.0 s, +20.0 s]`; Note: in SPM12 `Batch_1st_analysis`, the `Q` regressor of no interest was modeled at scan onset `15(b-1) + 8` = `+16.0` s with duration `2` scans = `4.0` s, covering the 1 s fixation + 3 s report window)
     - `10.0` s rest fixation (`[+20.0 s, +30.0 s]`).

### 4.2 Condition Codes & Session Sequences (`1234`, `2341`, `3412`, `4123`, `1123`)
There are **5 experimental conditions** (4 blocks per condition in each 20-block run):
- Code `1` (`0_ms`): **0 ms delay** (congruent gaze-contingent window, `controllable = 1`)
- Code `2` (`100_ms`): **100 ms delay** (`controllable = 1`)
- Code `3` (`300_ms`): **300 ms delay** (`controllable = 1`)
- Code `4` (`1000_ms`): **1000 ms delay** (`controllable = 1`)
- Code `5` (`Incongruency`): **Incongruent (uncontrollable) condition** (`controllable = -1`; spatial window mirrored across X or Y axis + pseudo-random 300 ms or 1000 ms delay on each trial).

The 4 session sequence templates (defined in `trial.c` line 190 and `SPM12_Conditions_per_session.xlsx`) specify the order of the 20 blocks (`1`..`5`) in a run:
- **Sequence 1**: `1,2,3,4,5, 2,1,5,4,3, 3,1,4,2,5, 5,1,3,2,4`
- **Sequence 2**: `4,1,3,5,2, 2,5,3,4,1, 1,2,3,4,5, 1,5,4,3,2`
- **Sequence 3**: `2,5,3,1,4, 4,2,1,3,5, 5,1,4,2,3, 3,5,2,4,1`
- **Sequence 4**: `1,2,3,4,5, 5,4,2,1,3, 3,2,4,1,5, 5,1,4,3,2`
- *(Practice Sequence 5)*: `2,5,3,1,4, 4,3,5,2,1, 1,2,3,4,5, 5,3,2,1,4`

Note: A participant's 4-digit sequence (e.g., `sub-01` = `3412`) means:
- `run-01` (`BOLD_DESCEND_1`, `<Initial>1 Result Data.csv`) used **Sequence 3**
- `run-02` (`BOLD_DESCEND_2`, `<Initial>2 Result Data.csv`) used **Sequence 4**
- `run-03` (`BOLD_DESCEND_3`, `<Initial>3 Result Data.csv`) used **Sequence 1**
- `run-04` (`BOLD_DESCEND_4`, `<Initial>4 Result Data.csv`) used **Sequence 2**
*(In `<Initial><1..4> Result Data.csv`, the file number `1..4` is always the chronological fMRI run order `run-01..run-04`, and the actual block conditions are recorded row-by-row in `Temporal delay condition(msec)` and `controllable`!)*

### 4.3 Behavioral CSV Columns (`<Initial><1..4> Result Data.csv`)
Each CSV has 100 rows (20 blocks × 5 trials):
1. `Total trial number`: `1` to `100`
2. `block number`: `1` to `20`
3. `trial number`: `1` to `5` (within block)
4. `Temporal delay condition(msec)`: `0`, `100`, `300`, or `1000` (Note: for Incongruency blocks where `controllable == -1`, this column records the trial-specific delay of `300` or `1000` ms)
5. `controllable`: `1` = delay condition (`0_ms`, `100_ms`, `300_ms`, `1000_ms`), `-1` = `Incongruency` condition
6. `Target existence`: `1` = Go trial (central character matches peripheral characters, 2 per block), `0` = No-Go trial (non-matching, 3 per block)
7. `presented stimuli index`: `0` to `99` (stimulus image index)
8. `Delay judgment`: Recorded on row `trial number == 5` of each block (`0` on rows 1–4):
   - **`2`** = **Self-initiated / No delay** (`self`, green button)
   - **`3`** = **Self-initiated but delayed** (`delay`, yellow button)
   - **`4`** = **Manipulated by computer / Other** (`other`, red button)
   - **`-1`** or **`0`** = Missing / no response within 3 s window
   - *(Note: For `GK2`, `GK3`, `HD3`, `LD1`, `LD2`, `LD4`, `MT3`, `MT4`, `NT1`, check `behavior_and_eyetracking/processed_summaries/MRI_combined.xlsx` for any post-EDF-verified subjective report corrections.)*
9. `target report`: `1` = participant pressed Go button during the 2 s trial, `-1` = no press
10. `target report RT`: Reaction time in ms (note: in `trial.c`, `totalmsec` includes initial fixation/mask time; see `MRI_combined.xlsx` for cleaned RT)
11. `(ISI) target report` & `(ISI) target report RT`: Button press and RT if pressed during the subsequent 1.5 s ISI / 1 s waiting window
12. `Type of uncontrol`: `-1` for controllable blocks; `1` = X-axis mirror, `2` = Y-axis mirror during Incongruency blocks
13. `participantEYE_arrive_time(include mask time)`: Gaze arrival time at peripheral character ROI (ms)
14. `participantEYE_saccade_target`: Saccade target location (`0` = top_left, `1` = top_right, `2` = bottom_left, `3` = bottom_right)

---

## 5. EyeLink Eye-Tracking Data (`behavior_and_eyetracking/`)

- Raw EyeLink 1000 Plus files (`.EDF`, 1000 Hz monocular recording) for all 23 participants × 4 fMRI runs (`<EyeLinkInitial>1.EDF` .. `<EyeLinkInitial>4.EDF`) are in `behavior_and_eyetracking/fMRI_main/`, and practice runs (`<EyeLinkInitial>5.EDF`) are in `behavior_and_eyetracking/fMRI_practice/`.
- See `info/Eye_movements_spec.docx` for exact EDF trigger and message markers (`MRI_TTL_Trigers`, `WAITKEY 116`, `BlockStart TemporalDelayCondition`, `TrialDisplayOn`, `TrialEnd`, `ReportStart`, `ReportEnd`, `EYE_USED`).
- Processed eye-movement distributions (fixation duration and saccade amplitude histograms used in Figure 4C–D of `Method.pdf`) and the Python parsing scripts are in `behavior_and_eyetracking/processed_summaries/`.

---

## 6. SPM12 & LOSO Derivatives (`derivatives_spm12/` & `derivatives_loso/`)

1. **Subject-Level Preprocessing & 5-Condition GLM (`derivatives_spm12/preproc_and_1st_level_5cond/sub_XX/`)**:
   - `BOLD_DESCEND_1..4/rp_*_0006.txt`: 6 rigid-body realignment parameters (3 translations in mm, 3 rotations in radians) across 300 volumes -> maps to BIDS `desc-confounds_timeseries.tsv`.
   - `BOLD_DESCEND_1/mean*_0006.nii`: Mean functional image after realignment.
   - `BOLD_DESCEND_1..4/swar*_0006..0305.nii`: Final preprocessed (realigned `r`, slice-timed `a`, MNI-normalized 2×2×2 mm `w`, 8 mm FWHM smoothed `s`) 3D NIfTI volumes.
   - `T1/w*.nii` & `T1/y_*.nii`: MNI-normalized anatomical T1w and forward deformation field.
   - `Results/`: 1st-level GLM with 5 task regressors (`0_ms`, `100_ms`, `300_ms`, `1000_ms`, `Incongruency`), 1 questionnaire regressor (`Q`), and 6 motion regressors per session (4 sessions × 12 = 48 + 4 session constants = 52 columns in `SPM.mat`, plus contrasts `con_0001..0022.nii` and `spmT_0001..0022.nii` described in `info/Nii_Batching_README_contrasts.docx`).
2. **Subject-Level Parametric Modulation GLM (`derivatives_spm12/1st_level_parametric/sub_XX/Results_PM/`)**:
   - Models `Delayed` condition with a linear parametric modulator (delay in ms: 0, 100, 300, 1000), `Incongruency` condition, `Q`, and 6 motion parameters per session (`Method.pdf` Section 2.10.1).
3. **Group-Level 2nd-Level Analyses (`derivatives_spm12/2nd_level_group/`)**:
   - `Batch_2nd_analysis/PM_One_Sample_Cong_x_T/`: Delay-Modulated one-sample t-test (`Method.pdf` Section 2.10.1).
   - `Batch_2nd_analysis/Factorial_ANOVA_equalVAR_minus_INCONG/`: Delay-Related repeated-measures 1-way ANOVA on `(0, 100, 300, 1000 ms) - Incongruency` contrasts (`Method.pdf` Section 2.10.2).
   - `Batch_2nd_analysis/One_Sample_Incong_minus_0/`: `Incongruent - 0 ms` one-sample t-test (`Method.pdf` Section 2.10.3).
   - `SwE/`: Repeated-Measures Sandwich Estimator toolbox group analysis.
4. **Leave-One-Subject-Out (LOSO) Analyses (`derivatives_loso/`)**:
   - `LOSO_rSMG/`: 23-fold leave-one-subject-out ANOVA models (`LOSO_sub_01`..`LOSO_sub_23`), MarsBaR ROI definitions for right supramarginal gyrus (`rSMG`), MATLAB extraction scripts (`part_1.m`, `part_2_1.m`, `part_2_2.m`, `mat_to_Excel.m`), and extracted beta tables (`Processed_LOSO_results.xlsx`).
   - `LOSO_Angular_Gyrus/`: Corresponding 23-fold LOSO analysis and MarsBaR ROI definitions for Angular Gyrus (`Processed_LOSO_AG_results.xlsx`).
