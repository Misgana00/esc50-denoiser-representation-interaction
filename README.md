# Denoiser–Representation Interaction in Noise-Robust ESC

Code and per-run results accompanying:

> Endashaw, M., Yoseph, E., Getnet, K., Damtew, M., Zyout, I. "Denoiser–Representation Interaction in Noise-Robust ESC: A Systematic Study of Classical and Learned Enhancement Methods." *International Journal of Artificial Intelligence (IJ-AI)*, under revision.

This repository evaluates four denoising strategies — MMSE-LSA, adaptive Wiener filtering,
and a learned U-Net spectral-mask denoiser (compared against unprocessed noisy audio as the
baseline) — across **three architecturally distinct** environmental sound classification
pipelines on ESC-50, under additive white Gaussian noise (AWGN) and real-world traffic noise
at SNR levels from −10 dB to 10 dB.

| Pipeline | Representation | Classifier | Folder |
|---|---|---|---|
| 1 | Raw waveform | 1D CNN (trained from scratch) | [`pipelines/1d_cnn`](pipelines/1d_cnn) |
| 2 | MFCC + deltas | Subspace Discriminant Ensemble | [`pipelines/mfcc_ensemble`](pipelines/mfcc_ensemble) |
| 3 | Log-mel spectrogram | CNN14 (AudioSet-pretrained) | [`pipelines/cnn14`](pipelines/cnn_14) |

All statistical results reported in the paper (Tables 4–6: recovery, 95% CI, Cohen's *d*z,
paired *t*-tests, BH-FDR / Holm-Bonferroni correction) are recomputed directly from the raw
per-run CSVs in [`results/per_run_csv`](results/per_run_csv) by
[`analysis/statistical_analysis_reproducibility.ipynb`](analysis/statistical_analysis_reproducibility.ipynb) —
no number in the manuscript is hand-entered.

---

## Repository layout

```
pipelines/
  1d_cnn/
    01_train_and_mmse_baseline.ipynb   # trains the 1D CNN; evaluates MMSE-LSA and Butterworth/spectral-subtraction baselines
    02_unet_denoiser_multirun.ipynb    # trains the U-Net (FFT=512, hop=128); 5-run noise-robustness eval; loads the 01_ checkpoint
  mfcc_ensemble/
    01_train_and_wiener_baseline.ipynb # extracts MFCCs, trains the ensemble; evaluates the adaptive Wiener filter
    02_unet_denoiser_multirun.ipynb    # trains the U-Net (FFT=1024, hop=256, base_ch=32); 5-run noise-robustness eval
  cnn14/
    01_train_and_mmse_baseline.ipynb   # fine-tunes CNN14 (3-phase schedule); evaluates MMSE-LSA (FFT=1024, hop=320)
    02_unet_denoiser_multirun.ipynb    # trains the U-Net (FFT=512, hop=128, matches Pipeline 1); 5-run noise-robustness eval

analysis/
  statistical_analysis_reproducibility.ipynb   # recomputes Tables 4-6 from results/per_run_csv, no hand-entered numbers

results/
  per_run_csv/          # raw per-run accuracy, five seed-matched runs per condition (n=5)
    gaussian_unet_per_run.csv     # 1D CNN, Gaussian noise
    traffic_unet_per_run.csv      # 1D CNN, traffic noise
    mfcc_unet_multirun_raw.csv    # MFCC ensemble, Gaussian + traffic
    traffic_unet_multirun_raw.csv # CNN14, Gaussian + traffic (+ white, not reported in the paper)
  reproduced_tables_4_6.csv       # output of analysis/, diff this against the manuscript tables

data/
  README.md              # how to obtain ESC-50 and the traffic-noise recording (not redistributed here)

requirements.txt
environment.yml
LICENSE
CITATION.cff
```

### Run order

Each pipeline's `02_` notebook loads a checkpoint saved by its `01_` notebook
(`load_model(...)` / equivalent) — run `01_` first within a pipeline. The three pipelines
are independent of each other and can be run in any order or in parallel.
`analysis/statistical_analysis_reproducibility.ipynb` only needs the CSVs in
`results/per_run_csv/` and does not require re-running any training notebook.

---

## Setup

```bash
# Option A: pip
pip install -r requirements.txt

# Option B: conda
conda env create -f environment.yml
conda activate esc-denoise
```

Core dependencies: `torch`, `torchaudio`, `tensorflow`, `librosa`, `scikit-learn`,
`pandas`, `numpy`, `scipy`, `seaborn`, `matplotlib`, `tqdm`.

Notebooks were developed on Google Colab (NVIDIA Tesla T4 GPU); Drive-mount cells at the top
of each notebook can be skipped when running locally — just point `DATA_PATH` at a local
copy of ESC-50 (see [`data/README.md`](data/README.md)).

---

## Data

This repository does **not** redistribute audio. `data/README.md` gives:
- the ESC-50 download link and expected folder layout (`audio/`, `meta/esc50.csv`);
- the exact Freesound.org source for the traffic-noise recording used in all traffic-noise
  experiments ("Crowd traffic under bridge in Cairo", Freesound user `34mwi_`), plus the
  preprocessing steps (M4A→WAV, resample, 5 s segmentation) needed to reproduce it.

---

## Reproducing the paper's tables

1. Run each pipeline's `01_` then `02_` notebook (or use the CSVs already committed under
   `results/per_run_csv/` to skip straight to step 2).
2. Open `analysis/statistical_analysis_reproducibility.ipynb` and run all cells.
3. Compare `results/reproduced_tables_4_6.csv` against Tables 4, 5, and 6 in the manuscript.

Evaluation protocol matches manuscript §3.2: 5 seed-matched runs per condition, paired
two-sided *t*-test, Cohen's *d*z with 95% CI (*t*-distribution, df = 4), multiple-comparisons
correction via Benjamini-Hochberg FDR (primary) and Holm-Bonferroni (conservative secondary
check) across all 30 reported conditions.

---

## Results at a glance (clean-audio baseline, sanity check)

Use this to confirm your environment is set up correctly before running noise experiments —
these are the Table 3 numbers, 5-fold CV mean ± std on clean ESC-50:

| Model | Accuracy |
|---|---|
| 1D CNN | 48.34% ± 4.8 |
| MFCC Ensemble | 54.00% ± 0.74 |
| CNN14 (PANNs) | 86.95% ± 2.5 |

## Hardware / runtime notes

All notebooks were run on Google Colab with an NVIDIA Tesla T4 GPU. Approximate wall-clock
time per pipeline (`01_` + `02_` notebooks combined, including 5-run noise-robustness
evaluation): CNN14 is the slowest to fine-tune (three-phase schedule, 75 epochs total);
1D CNN and MFCC train faster from scratch but the MFCC augmentation step (1,600 → ~14,400
samples) adds noticeable preprocessing time. A CPU-only machine will run, but the U-Net
denoisers will be considerably slower at both training and inference (see Table 7 in the
paper for the GPU-vs-CPU latency discussion).

## Known limitations

- Evaluated on ESC-50 only; UrbanSound8K, SONYC-UST, and DCASE are noted as future work in
  the manuscript, not evaluated here.
- CNN14 is AudioSet-pretrained while the 1D CNN and MFCC ensemble are trained from scratch;
  cross-pipeline accuracy comparisons should be read with that difference in mind.
- Real-world noise experiments use a single traffic-noise recording (Freesound, Cairo);
  results may not generalize to other traffic recordings without re-validation.

## Authors / contact

Mhretewold Endashaw, Eden Yoseph, Kalkidan Getnet, Misgana Damtew, Imad Zyout —
Engineering Technology & Science, Higher Colleges of Technology, Abu Dhabi, UAE.
Corresponding author: Imad Zyout (izyout@hct.ac.ae).
For questions about this repository specifically, open a GitHub issue.

## Citation

If you use this code, please cite the paper (see [`CITATION.cff`](CITATION.cff)). A citable
release (with DOI, e.g. via Zenodo) will be added once the manuscript is accepted —
see the note under "Releases" below.

## License

Code: [MIT License](LICENSE) 
Data: not included; see `data/README.md` for original source licenses (ESC-50, Freesound).
