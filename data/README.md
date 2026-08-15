# Data

No audio is committed to this repository. Both required sources are freely available;
below is exactly what each pipeline expects on disk.

## 1. ESC-50

- Source: https://github.com/karoldvl/ESC-50 (Piczak, 2015)
- Download and unzip so you have:
  ```
  ESC-50-master/
    audio/            # 2,000 .wav clips
    meta/esc50.csv     # filename, fold, target, category, ...
  ```
- Point `DATA_PATH` in each pipeline's `01_` notebook at the folder containing
  `ESC-50-master/`.
- License: ESC-50 audio is a mix of individually licensed Freesound clips; see the
  dataset's own repository for per-clip attribution.

## 2. Traffic noise recording

- Source: Freesound.org, "Crowd traffic under bridge in Cairo" by user `34mwi_`
  https://freesound.org/people/34mwi_/sounds/851965/
- Original file: mono M4A, 48,000 Hz, ~17.7 s.
- Preprocessing applied before use (see `01_` notebooks' noise-loading cells):
  1. Convert M4A → WAV.
  2. Resample to match the ESC-50 pipeline's working sample rate (22,050 Hz for the 1D
     CNN pipeline; 44,100 Hz for MFCC; 32,000 Hz for CNN14 — see each pipeline's `SR`
     constant).
  3. Segment into 5 s clips to match the ESC-50 clip duration.
- Save the processed file as `traffic_noise.wav` under your `DATA_PATH` (the exact
  filename each notebook expects is set by `REAL_NOISE_PATH` / equivalent near the top
  of the `01_` notebook).
- License: check the Freesound page for the specific Creative Commons license attached
  to this recording before redistributing it yourself.

## Directory expected by the notebooks

```
DSP Project/                       
  esc50_data/ESC-50-master/
    audio/
    meta/esc50.csv
  traffic_noise.wav
```
