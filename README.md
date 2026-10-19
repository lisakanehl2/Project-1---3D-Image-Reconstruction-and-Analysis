# HL2027 Project 1 – Radial sampling in MRI

Group 2: Lisa Kanehl, Valeria Sabas Ortega

Everything is in **`project1.ipynb`**, one notebook that goes through Tasks 1–7 in order:

| Task | Content |
|---|---|
| 1 | Radial trajectory (`sigpy.mri.radial`, no golden angle), simulated k-space with the forward NUFFT, acceleration factor R = 256 / spokes |
| 2 | PSNR, SSIM, NRMSE, NMI and difference maps, tested on simple cases first |
| 3 | Gridding (adjoint NUFFT) without DCF, with ramp and Pipe–Menon DCF, CG least squares, standard inverse FFT (nearest grid and Cartesian), Nyquist check, sigpy tutorial data |
| 4 | Compressed sensing (`L1WaveletRecon`, `TotalVariationRecon`): λ, DCF weights, iterations, wavelet type, noise |
| 5 | Sinogram directly from the radial k-space (pairing opposite spokes and conjugate symmetry), check against the Radon transform, odd vs. even numbers of spokes |
| 6 | Filtered back projection (5 filters) and SART |
| 7 | Slice-by-slice 3D reconstruction at R ≈ 5 and R ≈ 10 with the best method, saved as NIfTI |

## Run

```bash
pip install -r requirements.txt
jupyter notebook project1.ipynb
```

`t1.nii`, `projection_ksp.npy` and `projection_coord.npy` must be in the same folder.
The whole notebook takes about 30 minutes on 2 cores (Task 4 and Task 7 are the slow parts; set `N_CPU` at the top).

## Output

- `figures/` – all figures as PNG and PDF
- `results/` – tables (CSV) and the 3D reconstructions (`task7_*.nii.gz`, open in ITK-SNAP)
