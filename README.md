# 2P Calcium Imaging Cell Extraction Pipeline

One-run MATLAB pipeline for **2-photon calcium imaging**: TIF → motion correction → denoising → EXTRACT (ROI detection) → optional curation (ActSort / SignalSorter) → ΔF/F and figures.

## Overview

- **Input:** Multi-frame TIF stack (e.g. from miniscope/2P).
- **Output:** Motion-corrected HDF5 movie, EXTRACT ROIs (spatial + temporal weights), ΔF/F traces, overlay/heatmap figures, optional QC report and check videos.

Pipeline follows the [EXTRACT user manual](https://github.com/schnitzer-lab/EXTRACT-public) workflow and is compatible with **ActSort** for post-extraction cell sorting.

## Requirements

- **MATLAB** (tested R2021b+).
- **EXTRACT:** [schnitzer-lab/EXTRACT-public](https://github.com/schnitzer-lab/EXTRACT-public).
- **NoRMCorre:** for motion correction.
- **Optional:** ActSort ([ActSort-public](https://github.com/ActSort/ActSort-public)), CIAtah (for SignalSorter).

## Setup

1. Clone this repo and add to MATLAB path (or `cd` to it).
2. Install EXTRACT and NoRMCorre; clone ActSort if you use it.
3. In `speed_dff_extract_HS_tiff.m`, set:
   - `EXTRACT_path`, `NoRMCorre_path`, `ActSort_path` (if used)
   - `output_folder`, `curr_header`
   - `file_path`, `file_name` (input TIF)

## Quick start

1. Open `speed_dff_extract_HS_tiff.m` in MATLAB.
2. Set paths and input/output (see top of script).
3. Run with **F5** (or `run('speed_dff_extract_HS_tiff.m')`).

- First run: `skip_steps_1_2 = false` (full pipeline).
- Re-run only EXTRACT (e.g. after tuning): `skip_steps_1_2 = true` (loads from saved HDF5).

## Main options (in script)

| Option | Description |
|--------|-------------|
| `skip_steps_1_2` | `true` = load denoised from HDF5, skip load + motion correction + denoise |
| `extract_mode` | `'step1'` / `'step2'` / `'final'` — EXTRACT manual tuning vs full run |
| `fast_detect_mode` | `true` = temporal downsampling inside EXTRACT only (faster; traces still full-rate) |
| `run_actsort_precompute` | Build precomputed file for ActSort (EXTRACT .mat + H5) |
| `run_qc_report` | Save QC_report.mat / .txt / .png |
| `n_frames_check` | Number of frames in check videos (0 = skip) |

## Outputs

- **HDF5:** `*_1.h5` — motion-corrected, denoised movie (`/mov`).
- **EXTRACT:** `*_MC_extractout.mat` — `output` (spatial_weights, temporal_weights).
- **Figures:** cell overlay, ΔF/F heatmap, trace plots, EXTRACT cell map.
- **Optional:** QC report, check videos, ActSort precomputed file.

## Documentation

- **[PROTOCOL.md](PROTOCOL.md)** — Full protocol: pipeline steps, EXTRACT parameters, morphology cleanup, ActSort/SignalSorter, and troubleshooting.

## Author

Pipeline developed for 2P/miniscope calcium imaging (e.g. GRIN lens).  
Repository: [limserenahansol](https://github.com/limserenahansol).
