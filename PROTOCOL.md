# Cell Extraction Protocol — Full Explanation

This document describes the pipeline implemented in `speed_dff_extract_HS_tiff.m`: steps, parameters, and how to tune them.

---

## 1. Pipeline Steps

| Step | What it does |
|------|----------------|
| **1. Load** | Reads all TIF frames (no temporal downsampling by default). Optional check video: `01_check_after_downsampling.avi`. |
| **2. Motion correction + denoise** | NoRMCorre → per-frame spatial Gaussian (σ=1) → save to HDF5 `*_1.h5`. Optional: `03_check_after_denoising.avi`. |
| **3. EXTRACT** | Spatial bandpass (highpass 5 px, no lowpass) → EXTRACT cell finding + refinement → NaN removal → morphology/dynamics cleanup → edge rejection. Saves `*_MC_extractout.mat`. Optional: `04_check_after_EXTRACT_overlay.avi`. |
| **3b. ActSort precompute** | If enabled: runs `PrecomputeCellCheck(extract_mat, h5_file)` so ActSort can open the session. Requires dense `double` spatial/temporal weights (script enforces this). |
| **4. SignalSorter** | Optional CIAtah signalSorter for automated accept/reject. If it returns 0 cells, script falls back to raw EXTRACT output. |
| **5. ΔF/F and figures** | Baseline = mean trace; ΔF/F = (F−F0)/F0_safe. Saves overlay, heatmap, stacked traces, EXTRACT cell map. |

---

## 2. EXTRACT Configuration (summary)

- **Preprocessing:** Script applies `spatial_bandpass()` externally; EXTRACT is given `preprocess = 0`, `skip_dff = 1`, `F_per_pixel` constant so bandpass is not applied twice.
- **Cell size:** `avg_cell_radius = 8` (tune for your FOV/zoom).
- **Detection:** `cellfind_max_steps`, `cellfind_min_snr`, `cellfind_numpix_threshold`, `init_with_gaussian` — tuned to allow dim/small neurons while limiting junk.
- **Shape/size:** `eccent_thresh`, `spatial_corrupt_thresh`, `size_upper_limit`, `size_lower_limit`.
- **Trace quality:** `T_min_snr`, `low_ST_index_thresh`, `low_ST_corr_thresh`.
- **Overlap:** `S_dup_corr_thresh`, `T_dup_corr_thresh`.
- **Refinement:** `max_iter = 6`, partitions 2×2.

**Fast detection mode:** `fast_detect_mode = true` uses `downsample_time_by` (e.g. 4) only inside EXTRACT; final traces and saved movie remain full frame rate.

---

## 3. Morphology + Dynamics Cleanup (post-EXTRACT)

After EXTRACT, ROIs are filtered so that only soma-like, dynamic cells are kept:

- **Area:** ≥ `min_area_px` (e.g. π×(0.45×radius)²).
- **Shape:** Eccentricity ≤ 0.95, Solidity ≥ 0.45.
- **Dynamics:** Peak ΔF/F (99th percentile, baseline = 20th) ≥ `min_peak_dff` (e.g. 0.05) to drop static noise.
- **Edge:** Centroids must be at least `avg_cell_radius` pixels away from FOV borders.

---

## 4. EXTRACT Manual Modes (`extract_mode`)

- **`step1`:** Cell finding only (`max_iter = 0`, visualization on). Use to tune detection (e.g. `cellfind_min_snr`, radius).
- **`step2`:** One refinement + hyperparameter curves. Use to tune thresholds; then set `extract_mode = 'final'`.
- **`final`:** Full extraction (default).

---

## 5. ActSort Compatibility

- EXTRACT output is saved with **dense** `spatial_weights` and `temporal_weights` (no `ndSparse`), so ActSort’s `PrecomputeCellCheck` and `find_cell_centers` work.
- Run with `run_actsort_post` and `run_actsort_precompute = true` to generate `precomputed_*.mat` in the output folder. In ActSort: File → Open → select that precomputed file.

---

## 6. Output Files Reference

| File | Content |
|------|---------|
| `*_1.h5` | Motion-corrected, denoised movie (`/mov`) |
| `*_MC_extractout.mat` | `output` (spatial_weights, temporal_weights) |
| `*_MC_extractout_ps.mat` | Post–SignalSorter output (if run) |
| `analysis_results.mat` | Accumulated results (append) |
| `final_analysis_results.mat` | output, output_ps, deltaF_over_F, max_peaks, top_cells_indices |
| `cell_overlay_full_FOV.png` | Mean image + yellow contours |
| `deltaF_over_F_heatmap.png` | ΔF/F heatmap |
| `dff_traces_stacked_all_cells.png` | Stacked traces with cell IDs |
| `extract_cellmap.png` | EXTRACT native cell map |
| `QC_report.mat/.txt/.png` | Optional QC (when `run_qc_report = true`) |

---

## 7. Troubleshooting

- **Too few cells:** Loosen `cellfind_min_snr`, `T_min_snr`, `size_lower_limit`; lower `min_peak_dff` or `min_area_px`.
- **Too many / noise ROIs:** Tighten the above; raise `min_peak_dff` or `min_solidity`.
- **ActSort “invalid precomputed file”:** Ensure you run the script with precompute (so `precomputed_*.mat` exists) and open that file in ActSort, not the raw EXTRACT .mat.
- **Path conflicts (e.g. `get_circularity_metrics`):** Script adds ActSort only when calling `PrecomputeCellCheck` and re-adds EXTRACT first; if you use other toolboxes, check `addpath` order.

For more detail on EXTRACT parameters and tuning, see the [EXTRACT user manual](https://github.com/schnitzer-lab/EXTRACT-public).
