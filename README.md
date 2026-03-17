# 2P Calcium Imaging Cell Extraction Pipeline

One-run MATLAB pipelines for **2-photon calcium imaging**: TIF → motion correction → denoising → EXTRACT (ROI detection) → optional curation (ActSort / SignalSorter) → ΔF/F and figures.

This repository provides **two versions** of the same pipeline. **v2** combines the original single-file pipeline with features from a colleague’s preprocessing workflow (batch mode, custom motion-correction template, optional temporal binning, and stricter EXTRACT preset).

---

## Two versions

| Script | Description |
|--------|-------------|
| **`speed_dff_extract_HS_tiff.m`** | Original single-file pipeline: one TIF per run, simple MC, full-rate EXTRACT. Best for single sessions and straightforward runs. |
| **`speed_dff_extract_HS_tiff_integrated_v2.m`** | Integrated pipeline combining this repo’s pipeline with colleague’s code: **batch** processing, **custom MC template** from a stable block, optional **z-score** and **temporal binning** before EXTRACT, **permissive/stricter** EXTRACT presets, **full-length raw traces** even when EXTRACT is run on binned data, optional **per-cell** trace plots and **dual-channel** loading. Use for multi-file runs or when you need template-based MC or faster EXTRACT via binning. |

### Similarities (both versions)

- Same core steps: load TIF → motion correction (NoRMCorre) → spatial denoise → spatial bandpass → EXTRACT → NaN/morphology cleanup → optional ActSort precompute & SignalSorter → ΔF/F and figures.
- Same EXTRACT workflow and morphology/dynamics filters (area, eccentricity, solidity, peak ΔF/F, edge margin).
- Same outputs: HDF5 movie, EXTRACT `.mat`, cell overlay, ΔF/F heatmap, stacked traces, trace trajectory plots, optional QC report and step-wise check videos.
- Both support **full-length raw** ΔF/F traces and optional **per-cell** trace plots (`trace_trajectories_per_cell/`).

### Differences (v2 adds or changes)

- **Run mode:** `run_mode = 'single'` (one TIF) or `'batch'` (all TIFs in `input_dir`; one output folder per file).
- **Motion correction:** Optional **custom template** from a stable frame block (`use_custom_mc_template`, `frame_range`, `templ_len`) for difficult motion; configurable `mc_max_shift`.
- **Pre-EXTRACT:** Optional **pixel-wise z-score** (`use_zscore_before_extract`) and **temporal binning** (`extract_bin_time`). When binning is used, v2 **recomputes full-length traces** by projecting the raw denoised movie onto EXTRACT ROIs, so all plots and saved ΔF/F are at full frame rate.
- **EXTRACT preset:** `extract_preset = 'permissive'` (original, dim/small cells) or `'stricter'` (colleague-style, higher SNR).
- **Dual-channel:** Optional loading of odd/even frames as green/red (`dual_channel`); EXTRACT runs on green only.
- **Trace export:** Optional save of raw and Z-scored traces per session (`save_trace_files`).
- **Platform:** Paths switch by `ispc` (Windows vs Mac/Linux) in v2.

Use **v1** for one-off single files and minimal options. Use **v2** for batch runs, template-based MC, or when you want to try colleague-style preprocessing (z-score, binning, stricter EXTRACT) while keeping full-length traces and the same downstream outputs.

---

## Overview

- **Input:** Multi-frame TIF stack (e.g. from miniscope/2P).
- **Output:** Motion-corrected HDF5 movie, EXTRACT ROIs (spatial + temporal weights), ΔF/F traces (full-length raw when using v2 with binning), overlay/heatmap figures, optional QC report, check videos, and per-cell trace plots.

Pipeline follows the [EXTRACT user manual](https://github.com/schnitzer-lab/EXTRACT-public) workflow and is compatible with **ActSort** for post-extraction cell sorting.

## Requirements

- **MATLAB** (tested R2021b+).
- **EXTRACT:** [schnitzer-lab/EXTRACT-public](https://github.com/schnitzer-lab/EXTRACT-public).
- **NoRMCorre:** for motion correction.
- **Optional:** ActSort ([ActSort-public](https://github.com/ActSort/ActSort-public)), CIAtah (for SignalSorter).

## Setup

1. Clone this repo and add to MATLAB path (or `cd` to it).
2. Install EXTRACT and NoRMCorre; clone ActSort if you use it.
3. In the script you use (**v1** or **v2**), set:
   - `EXTRACT_path`, `NoRMCorre_path`, `ActSort_path` (if used)
   - **v1:** `output_folder`, `curr_header`, `file_path`, `file_name`
   - **v2:** For single run: same as v1; for batch: `input_dir`, `main_output_dir`, `file_pattern`

## Quick start

**v1 (single file)**  
1. Open `speed_dff_extract_HS_tiff.m`.  
2. Set paths and `file_path` / `file_name`.  
3. Run with **F5**. Use `skip_steps_1_2 = true` to re-run only EXTRACT from saved HDF5.

**v2 (single or batch)**  
1. Open `speed_dff_extract_HS_tiff_integrated_v2.m`.  
2. Set `run_mode = 'single'` or `'batch'` and the corresponding paths.  
3. Run with **F5**. Same `skip_steps_1_2` behavior for re-running EXTRACT only.

## Main options (v1)

| Option | Description |
|--------|-------------|
| `skip_steps_1_2` | `true` = load denoised from HDF5, skip load + MC + denoise |
| `extract_mode` | `'step1'` / `'step2'` / `'final'` — EXTRACT tuning vs full run |
| `fast_detect_mode` | `true` = temporal downsampling inside EXTRACT only (traces still full-rate) |
| `run_actsort_precompute` | Build precomputed file for ActSort |
| `run_qc_report` | Save QC_report.mat / .txt / .png |
| `n_frames_check` | Frames in check videos (0 = skip). Per-cell traces: `save_trace_trajectory_per_cell` |

## Main options (v2, in addition to v1-like)

| Option | Description |
|--------|-------------|
| `run_mode` | `'single'` or `'batch'` |
| `use_custom_mc_template` | Build MC template from stable block (`frame_range`, `templ_len`) |
| `use_zscore_before_extract` | Pixel-wise z-score before EXTRACT |
| `extract_bin_time` | Temporal bin factor for EXTRACT input (1 = full rate). Traces are still full-length (projection). |
| `extract_preset` | `'permissive'` (dim cells) or `'stricter'` (colleague-style) |
| `save_trace_trajectory_per_cell` | Save one trace plot per cell in `trace_trajectories_per_cell/` |
| `save_trace_files` | Save raw + Z traces per session (`*_traces_raw_Z.mat`) |
| `dff_trace_source` | `'extract'` (default) = ΔF/F traces for **all** EXTRACT+morph cells (matches overlay). `'signal_sorter'` = only SignalSorter-accepted cells (often very few if `automate=1`). |

## Outputs (both versions)

- **HDF5:** `*_1.h5` — motion-corrected, denoised movie (`/mov`).
- **EXTRACT:** `*_MC_extractout.mat` — `output` (spatial_weights, temporal_weights).
- **Figures:** cell overlay, ΔF/F heatmap, stacked traces, trace trajectories, EXTRACT cell map.
- **Optional:** QC report, step check videos (01–04), `trace_trajectories_per_cell/` (one PNG per cell), ActSort precomputed file.  
- **v2 only (optional):** `*_traces_raw_Z.mat` when `save_trace_files = true`.

## Documentation

- **[PROTOCOL.md](PROTOCOL.md)** — Full protocol: pipeline steps, EXTRACT parameters, morphology cleanup, ActSort/SignalSorter, and troubleshooting.

## Author

Pipeline developed for 2P/miniscope calcium imaging (e.g. GRIN lens).  
**v2** integrates features from a colleague’s preprocessing pipeline (batch, custom MC template, z-score/bin options, stricter EXTRACT preset).  
Repository: [limserenahansol](https://github.com/limserenahansol).
