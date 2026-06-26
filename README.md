# MAPS-ASR

### End-to-End Robust ASR Based on Multi-Modal Noise Detection and Parameter-Efficient Accent Adaptation

**Status: Under journal submission**

---

## Overview

MAPS is a complete, deployable speech recognition pipeline that addresses two key challenges in real-world ASR: environmental noise and speaker accent. Rather than handling these problems separately, MAPS integrates three tightly coupled modules into a single end-to-end system.

Incoming audio is first screened for noise severity by an audio-visual fusion model. If clean enough, it is then classified by accent and routed to a lightweight LoRA-adapted Whisper model trained specifically for that speaker's phonetic patterns — all without modifying the base model.

---

## Key Results

### Module 1 — Audio-Visual Noise Detector

| Metric | Value |
|---|---|
| **Model size** | 1.67M parameters (~6.4 MB) |
| **Detection accuracy** | 99.4% |
| **ROC-AUC** | 0.9995 |
| **Inference time** | 2.65 ms per sample |
| **Test conditions** | 0–5 dB SNR (severe noise) |

Attention-based audio-visual fusion outperforms audio-only baseline by **+12.2 percentage points**.

### Module 2 — Accent Classifier

| Metric | Value |
|---|---|
| **Accuracy** | 99.9% (2,882/2,885 correct) |
| **Accents** | Vietnamese, Chinese, Korean, Indian English, Native English |

### Module 3 — LoRA-Adapted Whisper ASR

| Model | WER | Parameters Modified |
|---|---|---|
| Whisper-Small (baseline) | 24.91% | — |
| Whisper-Medium (baseline) | 12.32% | 100% |
| **MAPS (LoRA-Small)** | **10.08%** | **0.36%** |

LoRA-adapted Whisper-Small **outperforms Whisper-Medium** while modifying only 0.36% of parameters. Each accent adapter adds just 5.32 MB of storage.

### Per-Accent WER Results

| Accent | Whisper-Small | MAPS (LoRA) | Improvement |
|---|---|---|---|
| Vietnamese | 25.21% | 16.35% | −35.1% relative |
| Chinese | 17.71% | 11.44% | −35.4% relative |
| Korean | 11.28% | 7.29% | −35.4% relative |
| Indian | 8.23% | 5.32% | −35.4% relative |
| **Overall** | **24.91%** | **10.08%** | **−35.4% relative** |

All improvements are statistically significant: **p < 10⁻¹³** (paired t-tests), Cohen's d = 0.29–0.50.

---

## System Architecture

```
Input Audio (+optional lip video)
        │
        ▼
┌─────────────────────────────┐
│  Module 1: Noise Detector   │  Audio-visual CNN + BiLSTM + Attention
│  (1.67M params, 2.65ms)     │  → Clean / Too Noisy
└─────────────────────────────┘
        │ (if clean)
        ▼
┌─────────────────────────────┐
│  Module 2: Accent Classifier│  CNN on mel-spectrogram
│  (99.9% accuracy)           │  → Vietnamese / Chinese / Korean / Indian / Native
└─────────────────────────────┘
        │
        ▼
┌─────────────────────────────┐
│  Module 3: LoRA-Whisper ASR │  Whisper-Small + accent-specific LoRA adapter
│  (0.36% params, 10.08% WER) │  → Transcript
└─────────────────────────────┘
```

### LoRA Adaptation

LoRA (Low-Rank Adaptation) injects small trainable matrices into frozen Whisper-Small attention layers:

- **Rank:** r = 8
- **Scaling:** α = 16  
- **Target layers:** Query and Value projections (all encoder + decoder layers)
- **Trainable params per adapter:** 884,736 (0.36% of base model)
- **Training time:** ~4.15 GPU-hours total across 4 adapters (NVIDIA Tesla T4)

---

## Datasets

| Dataset | Purpose | Size |
|---|---|---|
| **L2-ARCTIC** | Accent detection + LoRA training | 19,228 utterances, 4 Asian accents |
| **LibriSpeech clean-100** | Clean speech for noise detection | ~2,000 utterances |
| **UrbanSound8K** | Noise signals (0–5 dB SNR mixing) | 8,732 noise clips, 10 categories |
| **VVAD-LRS3** | Lip video for audio-visual fusion | 44,489 sequences, 64×64 px |
| **CommonVoice** | Native English reference | 789 utterances |

---

## Repository Contents

```
maps-asr/
├── paper/
│   └── maps_asr_journal.pdf    # Journal paper (available upon publication)
└── README.md
```

---

## Citation

This work is currently under journal submission. Citation details will be added upon acceptance.

```
Nguyen, Ngoc Thanh Thanh, Tran, Manh Son, Bui, Ngoc Dung, and Mai, Duc-Tho.
"End-to-End Robust ASR Based on Multi-Modal Noise Detection and
Parameter-Efficient Accent Adaptation." (Under submission, 2025)
```

---

## Author

**Nguyen Ngoc Thanh Thanh (Tammy)**
Lead Researcher & First Author — responsible for all experimental design, model architecture, training, and evaluation.

[LinkedIn](https://linkedin.com/in/ngoc-thanh-thanh-nguyen-68004740b) · [Email](mailto:tammynguyen0699@gmail.com) · [GitHub](https://github.com/tammynguyen6)
