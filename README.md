# HL2027 Project 1 – Radial sampling in MRI

Group 2: Lisa Kanehl, Valeria Sabas Ortega

MRI is slow because k-space is measured line by line. In this project we simulate **radial sampling** (spokes through the
centre of k-space) of a T1-weighted brain image at accelerations from 1× to 10×, reconstruct it with gridding (NUFFT),
compressed sensing and CT methods, compare all methods against the ground truth, and apply the best one to the full 3D
volume at 5× and 10×.

Everything is in **`project1.ipynb`**, one notebook that goes through Tasks 1–7 in order. Each task starts with its aim,
explains the method, runs the experiments and ends with our observations.

| Task | Content |
|---|---|
| 1 | Radial trajectory (`sigpy.mri.radial`, no golden angle), simulated k-space with the forward NUFFT, acceleration factor R = 256 / spokes |
| 2 | PSNR, SSIM, NRMSE, NMI and difference maps, tested on simple cases first |
| 3 | Gridding (adjoint NUFFT) without DCF, with ramp and Pipe–Menon DCF, CG least squares, standard inverse FFT (nearest grid and Cartesian), Nyquist check, sigpy tutorial data |
| 4 | Compressed sensing (`L1WaveletRecon`, `TotalVariationRecon`): λ, DCF weights, iterations, wavelet type, noise |
| 5 | Sinogram directly from the radial k-space (pairing opposite spokes and conjugate symmetry), check against the Radon transform, odd vs. even numbers of spokes |
| 6 | Filtered back projection (5 filters) and SART |
| 7 | Slice-by-slice 3D reconstruction at R ≈ 5 and R ≈ 10 with the best method (TV) and CG as baseline, saved as NIfTI |

## Run

```bash
pip install -r requirements.txt
jupyter notebook project1.ipynb      # then run all cells
```

`t1.nii`, `projection_ksp.npy` and `projection_coord.npy` must be in the same folder as the notebook.
The whole notebook takes about 35 minutes on 2 cores; Task 4 and Task 7 are the slow parts. The number of cores is set by
`N_CPU` at the top (parallel runs use `fork`, so on Windows the loops simply run one after another).

Tested with Python 3.13, sigpy 0.1.27, scikit-image 0.26, SimpleITK 2.5, NumPy 2.5, SciPy 1.18, matplotlib 3.11,
pandas 3.0 and PyWavelets 1.8. A full run from a fresh clone finishes without errors and reproduces all tables.

## Repository

| Path | What it is |
|---|---|
| `project1.ipynb` | the notebook (code, explanations and outputs) |
| `t1.nii` | T1-weighted volume, 256 × 256 × 90 (slice 45 is used for Tasks 1–6) |
| `projection_ksp.npy`, `projection_coord.npy` | sigpy tutorial data (12 coils, 96 spokes × 512 samples) |
| `figures/` | all figures as PNG; the notebook also writes PDF versions for the report |
| `results/` | all result tables as CSV (`all_metrics.csv` has every method at every R) |
| `requirements.txt` | Python packages |

The 3D reconstructions (`results/task7_*.nii.gz`, open them in ITK-SNAP) and the PDF figures are created when the
notebook runs; they are not stored in the repository because of their size.

## Main results (slice 45, PSNR in dB / SSIM)

| Method | R = 1 | R ≈ 5 | R ≈ 10 |
|---|---|---|---|
| Gridding, no DCF | 16.0 / 0.16 | 16.0 / 0.16 | 16.0 / 0.16 |
| Gridding, Pipe–Menon DCF | 24.2 / 0.61 | 20.4 / 0.39 | 18.8 / 0.29 |
| CG least squares | 32.8 / 0.77 | 22.9 / 0.42 | 20.4 / 0.31 |
| FBP (Hann) | 29.6 / 0.78 | 20.4 / 0.37 | 16.9 / 0.23 |
| SART | 29.5 / 0.81 | 21.8 / 0.42 | 19.7 / 0.31 |
| L1-wavelet | 30.3 / 0.89 | 23.0 / 0.48 | 20.1 / 0.33 |
| Total variation | 29.9 / 0.86 | **24.1 / 0.58** | **20.9 / 0.37** |

- Density compensation is essential for gridding; CG beats the best gridding by 1.6–4.3 dB for R ≥ 2.
- TV (λ = 0.01, DCF weights, 300 iterations) is the best method from R = 2 upwards.
- The sinogram built directly from k-space matches the Radon transform of the ground truth to 0.5 %.
- 3D (whole volume): SSIM 0.83 (TV) vs. 0.56 (CG) at R ≈ 5, and 0.59 vs. 0.42 at R ≈ 10.

## Choices worth knowing

- **Even numbers of spokes** with R ≥ target (50 spokes for 5×, 24 for 10×). With an odd number, the mirrored spokes of a
  real-valued image fill the gaps, which a real scan would not give (studied in Task 5).
- **Ramp DCF** uses Δk/8 for the k = 0 sample (the area of the centre disc shared by all spokes).
- **λ** is chosen by a fixed rule: the highest SSIM among settings within 0.5 dB of the best PSNR at R ≈ 5.
- **Every reconstruction** is scored after taking the magnitude and one least-squares brightness factor.
- The ground truth is only used for scoring and for one geometry check of the sinogram, never for reconstruction.
