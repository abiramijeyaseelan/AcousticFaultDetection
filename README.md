# VAAZHVU — Acoustic Fault Detection on Edge

> Edge AI system for industrial machine health monitoring. Runs fully on ESP32-S3 — no cloud, no server.

**85.3% classification accuracy at 0 dB SNR · False alarm rate < 6% · ₹2,000 BOM · Top 50 — TNWise 2026, Thanjavur**

---

## What it does

VAAZHVU mounts on industrial machines and continuously listens for acoustic anomalies. Every 0.5 seconds it extracts 70 features from the microphone signal, runs two models on-chip, and outputs a live health score on an OLED display — no internet connection required.

A 30-second adaptive baseline calibration means it works on any new machine without retraining.

---

## Hardware

| Component | Purpose |
|---|---|
| ESP32-S3 | Main MCU — inference + control |
| MEMS Microphone | Acoustic signal acquisition |
| I2C OLED Display | Live health score readout |
| UART | Debug and serial event logging |

Total BOM: **₹2,000**

---

## ML Pipeline

### Feature Extraction — 70 features per 0.5s frame

| Group | Count | Features |
|---|---|---|
| Time domain | 7 | RMS, ZCR, crest factor, kurtosis, skewness, peak, peak-to-peak |
| Frequency domain | 11 | Spectral centroid, rolloff, bandwidth, band energies |
| MFCC | 52 | MFCC + Δ + ΔΔ (mean and std) |

### Dual-model on-chip inference

```
Microphone → Feature Extraction (70 features)
                    ↓
        ┌───────────┴───────────┐
        ↓                       ↓
  Autoencoder              Classifier
  (anomaly score)      (fault type ID)
        └───────────┬───────────┘
                    ↓
            Health Score + Alert
            (I2C OLED + UART)
```

- **Autoencoder** — trained on normal machine sounds (MIMII dataset). Reconstruction error → continuous health score 0–100
- **Classifier** — trained on CWRU bearing fault dataset. Identifies fault type (normal / inner race / outer race / ball fault)
- **Total model size:** 32 KB TFLite (INT8 quantised)

### Decision logic
```c
if (classifier_confidence > 0.70 && classifier_says_anomaly) {
    flag_fault();        // LED alert + UART log
} else if (classifier_confidence < 0.70 && classifier_says_anomaly) {
    log_warning();       // monitor, don't alert yet
}
// health_score always updates regardless
```

---

## Results

| Metric | Value |
|---|---|
| Classification accuracy | **85.3%** at 0 dB SNR |
| False alarm rate | **< 6%** |
| Baseline calibration time | 30 seconds |
| Inference rate | 10 Hz |
| Model size (both models) | 32 KB |

Evaluated on **MIMII industrial machine benchmark** (fan, id_00, 0 dB SNR condition).

---

## Datasets used

- **MIMII** — Malfunctioning Industrial Machine Investigation and Inspection dataset. Fan sounds at 0 dB SNR for anomaly detection training.
- **CWRU Bearing Dataset** — Case Western Reserve University. Drive-end accelerometer at 12 kHz, 0HP load. Classes: normal, inner race fault, outer race fault, ball fault.

---

## Repository structure

```
vaazhvu-acoustic-fault-detection/
├── firmware/                  # ESP32-S3 Embedded C firmware
│   ├── main.ino
│   ├── feature_extractor.h
│   ├── autoencoder_model.h    # TFLite model as C header
│   ├── classifier_model.h
│   ├── scaler_params.h        # Normalisation constants
│   └── model_config.h         # Thresholds + decision logic
├── training/                  # Google Colab ML pipeline
│   └── PS006_AcousticFaultDetection_Colab.py
├── plots/
│   ├── confusion_matrix.png
│   ├── threshold_analysis.png
│   ├── feature_distributions.png
│   └── feature_importance.png
└── README.md
```

---

## How to run the training pipeline

1. Open `training/PS006_AcousticFaultDetection_Colab.py` in Google Colab
2. Set runtime to **T4 GPU** (Runtime → Change runtime type)
3. Upload `0_dB_fan.zip` from MIMII dataset to `My Drive/PS006_AcousticFault/data/`
4. Run all cells top to bottom
5. Download `PS006_FINAL_firmware_package.zip` — contains all `.h` headers ready for Arduino

## How to flash firmware

```bash
# Arduino IDE
# Board: ESP32S3 Dev Module
# Flash mode: QIO 80MHz
# Open firmware/main.ino and upload
```



## Tech stack

`ESP32-S3` `TFLite` `TinyML` `INT8 quantisation` `I2C` `UART` `Python` `librosa` `TensorFlow/Keras` `MIMII` `CWRU`
