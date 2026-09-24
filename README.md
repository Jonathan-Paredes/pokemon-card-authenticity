# 🃏 Pokémon Card Authenticity Detection

> Detecting counterfeit Pokémon cards from photographs using deep learning — with an emphasis on **leakage prevention, external validation, and honest failure analysis**.

[![Python](https://img.shields.io/badge/Python-3.10-blue.svg)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.5.1-red.svg)](https://pytorch.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Internal F1](https://img.shields.io/badge/Internal%20F1-0.978-blue)]()
[![Blind F1](https://img.shields.io/badge/Blind%20Test%20F1-0.984-brightgreen)]()
[![External Recall](https://img.shields.io/badge/External%20Recall-95.7%25-success)]()
[![Domain Shift](https://img.shields.io/badge/Real--world%20Performance-Collapses-orange)]()

---

## 📌 TL;DR

A computer vision system that classifies Pokémon cards as **authentic** or **counterfeit** from photographs.

**What works:**

| Test | N | Result |
|---|---:|---:|
| Internal held-out test | 558 | F1 = **0.978** |
| Blind test (unseen cards, same protocol) | 262 | F1 = **0.984**, 4 errors |
| External test (new expansion, same protocol) | 47 | Recall = **95.7%** |
| Cross-domain gap (iPhone ↔ OnePlus) | — | 0.019 |

**What doesn't work — and why it matters:**

| Test | N | Result |
|---|---:|---:|
| Webcam images | 29 | ~0.55 (chance level) |
| Marketplace listing photos | 20 | Real recall = **0.15** |

The model **generalizes to new cards** but **fails under new acquisition conditions** (different camera, lighting, background, compression). This is the central finding of the project.

---

## 🎯 Project Overview

Counterfeit trading cards can look extremely similar to authentic ones. Detection requires recognizing subtle visual cues in:

- Printing quality and typography
- Color and tonal characteristics
- Borders and cardstock texture
- Holographic patterns
- Illustration details
- Small visual inconsistencies

This project investigates whether a CNN can learn these differences **from photographs of physical cards** — and, more importantly, whether the resulting model **generalizes to real-world conditions**.

### The core question

> **Can a deep learning model distinguish authentic Pokémon cards from counterfeit cards, and does that knowledge generalize to cards and conditions that were not present during training?**

The answer is: **yes on new cards, no on new cameras.** This distinction is the main contribution.

---

## 🔬 What Makes This Project Different

Most card authentication projects report a single accuracy number. This project focuses on the **full pipeline**:

1. **Detects and corrects data leakage** — image-level splits are broken; card-level splits are mandatory.
2. **Corrects EXIF orientation** — an almost never discussed problem that silently destroys performance.
3. **Evaluates external generalization** — internal accuracy means nothing if the model only works on the training protocol.
4. **Analyzes failure modes** — Grad-CAM reveals what the model actually looks at.
5. **Reports domain shift honestly** — the model collapses under real-world conditions, and we prove it.

---

## 🏗️ Architecture & Pipeline

Physical Pokémon Cards
│
▼
Photograph Acquisition (iPhone + OnePlus)
│
▼
Dataset Organization (real/fake × 7–10 views)
│
▼
Data Quality & Leakage Analysis
│
▼
EXIF Orientation Correction
│
▼
Card-Level Train / Validation / Test Split
│
▼
Anti-Leakage Verification (5 checks)
│
▼
Preprocessing + Augmentation
│
▼
EfficientNet-B0 + Transfer Learning
│
▼
Internal Test Evaluation
│
▼
Blind Test (262 unseen cards)
│
▼
External Test (new expansion)
│
▼
Real-World Test (webcam + marketplace)
│
▼
Grad-CAM Interpretability
│
▼
Domain Shift Analysis

---

## 📊 Dataset

### Composition

| Metric | Value |
|---|---:|
| Physical cards | 76 |
| Real/Fake pairs | 38 |
| Total images | 2,570 |
| Cameras | 2 (iPhone, OnePlus) |
| Views per card | 7–10 |
| Languages | EN, ES |

### Class balance

| Class | Images | Physical cards |
|---|---:|---:|
| real | 1,282 | 38 |
| fake | 1,288 | 38 |
| **Total** | **2,570** | **76** |

The dataset is class-balanced at the **physical card level**, not the image level. This prevents the model from learning a trivial majority-class prior.

### Views captured

Each card was photographed from multiple angles and regions to cover different authenticity-relevant features:

| View | Description |
|---|---|
| `FRONT` | Full frontal view |
| `FRONT_ABOVE` / `FRONT_BELOW` | Small angle variations |
| `FLASH` | Captured with flash |
| `LOWER_LEFT` / `UPPER_LEFT` | Corner crops |
| `WATERMARK` | Watermark region close-up |
| `CHARACTERISTICS` | Card-specific feature |
| `ANGLE15` / `ANGLE45` / `ANGLE75` | OnePlus only |

### Known acquisition biases

- **Two cameras only**: iPhone + OnePlus. No webcams, DSLRs, or other smartphones.
- **Controlled capture conditions**: consistent backgrounds and lighting during acquisition.
- **Protocol consistency**: all cards followed the same photographic protocol.

These choices enabled clean training but introduce **domain bias** — analyzed in detail below.

---

## 🔐 Preventing Data Leakage

### The problem

Multiple photographs can belong to the same physical card. Splitting by image leaks information:

C032_REAL_01.jpg → TRAIN
C032_REAL_02.jpg → TRAIN
C032_REAL_03.jpg → TEST ❌ Same physical card!


The model can memorize card-specific features and appear to generalize.

### The solution

Split **by `card_id`** so that every physical card appears in exactly one split.

| Split | Cards | Images |
|---|---:|---:|
| Train | 48 | 1,554 |
| Validation | 12 | 372 |
| Test | 16 | 644 |
| **Total** | **76** | **2,570** |

### Verification (5 independent checks)

| Check | Result |
|---|---|
| `card_id` in >1 split | **0** |
| `filename` in >1 split | **0** |
| `physical_id` in >1 split | **0** |
| Identical `md5` across splits | **0** |
| Near-duplicates (32×32 perceptual hash) across splits | **0** |

All checks pass. The split is leakage-free.

---

## 🖼️ EXIF Orientation Correction

### The problem

Smartphone cameras store images in **landscape pixel orientation** and add an **EXIF Orientation tag** (6 or 8) telling viewers to rotate on display.

- **iPhone**: pixel data is landscape, `Orientation = 6`
- **OnePlus**: pixel data is already rotated, `Orientation = 1`

Loading with `cv2.imread()` (which ignores EXIF) makes iPhone images appear rotated 90°. If the model sees inconsistent orientations, it learns spurious features that don't transfer.

### The solution

Every image was passed through `PIL.ImageOps.exif_transpose()` once, saving corrected versions to `Dataset_imagenes_fixed/`.

| Stage | iPhone | OnePlus |
|---|---|---|
| Before | 589 landscape (EXIF=6) | 699 portrait (EXIF=1) |
| After | 589 portrait (EXIF=1) | 699 portrait (EXIF=1) |

Without this fix, ~50% of the dataset would arrive sideways to the model, and performance would collapse with no obvious explanation.

**This is one of the most under-documented failure modes in image classification projects.**

---

## 🤖 Model

**EfficientNet-B0**, ImageNet-pretrained, fine-tuned end-to-end.

| Parameter | Value |
|---|---|
| Architecture | EfficientNet-B0 |
| Pretrained | ImageNet-1k |
| Parameters | ~5.3M |
| Input size | 224 × 224 × 3 |
| Classes | 2 (real, fake) |
| Modified layer | `classifier[1]`: Linear(1280 → 2) |

### Training configuration

| Parameter | Value |
|---|---|
| Optimizer | AdamW |
| Learning rate | 1e-4 |
| Weight decay | 1e-4 |
| Scheduler | CosineAnnealingLR (T_max=30) |
| Loss | CrossEntropyLoss, label_smoothing=0.1 |
| Batch size | 64 |
| Max epochs | 30 |
| Early stopping | patience=7 |
| Mixed precision | Yes |
| Seed | 42 |
| Training time | ~15 min (RTX 3060) |

### Selection criterion

The best checkpoint was chosen by **cross-camera F1**:

score = (F1_iphone + F1_oneplus) / 2


This penalizes models that excel on one camera but fail on the other.

### Augmentation

Applied online during training with Albumentations:

| Transform | Rationale |
|---|---|
| `RandomResizedCrop(0.7–1.0)` | Different framings |
| `HorizontalFlip` | Cards are symmetric |
| `Rotate(±12°)` | Small rotation tolerance |
| `ColorJitter(0.35, 0.35, 0.35, 0.08)` | Camera color variation |
| `RandomGamma(75–125)` | Exposure variation |
| `GaussNoise(5–30)` | Sensor noise |
| `MotionBlur(3)` | Hand-shake |
| `ISONoise` | Low-light artifacts |

**Not used**: `VerticalFlip`, `RandomRotate90`, `Transpose`. Watermarks, text, and card layout have canonical orientation.

---

## 📈 Results

### 1. Internal test (558 images)

| Metric | Value |
|---|---:|
| Accuracy | 0.9785 |
| F1 | 0.9783 |
| AUC | 0.9811 |
| F1 iPhone | 0.9883 |
| F1 OnePlus | 0.9695 |
| Gap (iPhone vs OnePlus) | 0.0188 |

### 2. Blind test (262 unseen cards, same protocol)

| Metric | Value |
|---|---:|
| Accuracy | 0.9847 |
| F1 | 0.9841 |
| AUC | 0.9998 |
| Errors | **4 / 262 (1.5%)** |

**All 4 errors are Trainer cards**, not Pokémon:

| File | Card | Type | Confidence |
|---|---|---|---|
| C032 | Sacred Ash | Item / Trainer | 0.77 |
| C033 | Ethan's Adventure | Supporter / Trainer | 0.68 |
| C026 | Team Rocket's Venture Bomb | Item / Trainer | 0.66 |
| C029 | Team Rocket's Petrel | Supporter / Trainer | 0.63 |

Trainer cards have plain backgrounds and text-dominant layouts — less visual signal to discriminate.

**Cross-camera consistency** (each card was captured twice):

| Category | Count |
|---|---:|
| Both cameras correct | 127 |
| Only OnePlus correct | 3 |
| Only iPhone correct | 1 |
| Both cameras wrong | **0** |

Zero cases of "both wrong" — the model's errors are borderline, not systematic per card.

### 3. External test (47 images, new expansion)

All 47 cards are authentic, from an expansion the model has never seen. Captured with flash, varied angles, and partial framing.

| Metric | Value |
|---|---:|
| Recall (real) | **0.9574** (45/47) |
| False positives | 2 |
| Mean confidence | 0.904 |
| Median confidence | 0.927 |

**The 2 false positives**:

| File | Card | Type | P(fake) |
|---|---|---|---|
| C037 | Sacred Charm | Trainer | 0.80 |
| C044 | Mega Gengar EX | Pokémon (holo) | 0.44 |

C037 is again a Trainer card. C044 is a holographic EX Pokémon, where the model's confidence was near the decision boundary.

### 4. Cross-domain (iPhone ↔ OnePlus)

| Experiment | Source F1 | Target F1 | Drop F1 |
|---|---:|---:|---:|
| iPhone → OnePlus | 0.9730 | 0.8538 | **0.1192** |
| OnePlus → iPhone | 0.9126 | 0.8430 | **0.0696** |

Training with a single camera loses ~7–12 F1 points when tested on the other. Training with both cameras closes the gap to 0.019 (see internal test).

### 5. Holographic cards (never seen in training)

The model was never trained on holographic cards (7 pairs were excluded — insufficient sample size). Still:

| Category | Correct | Total | Accuracy |
|---|---:|---:|---:|
| Real + holo reverse | 30 | 30 | **100%** |
| Real + holographic | 6 | 6 | **100%** |
| Real + no holo | 98 | 98 | **100%** |

**The model does not use the hologram as a shortcut.**

### 6. Language

| Language | N | Accuracy |
|---|---:|---:|
| English | 254 | 0.984 |
| Spanish | 8 | 1.000 |

**The model does not rely on language-specific text.**

---

## 🔴 Critical Finding: Domain Shift in Real-World Conditions

All results above share the **same acquisition protocol** as training: iPhone + OnePlus, controlled framing.

### The real-world test

Two additional datasets were collected under realistic conditions:

| Test | N | Description |
|---|---:|---|
| **Webcam** | 29 | Captured with a laptop webcam (18 fake + 11 real) |
| **Marketplace** | 20 | Photos pulled from online seller listings (all real) |

### Results

| Test | N | Real recall | Fake recall | Overall |
|---|---:|---:|---:|---:|
| Blind (control) | 262 | 0.988 | 0.977 | 0.984 |
| External | 47 | 0.957 | — | — |
| **Webcam** | 29 | 0.545 | 0.556 | **0.552** |
| **Marketplace** | 20 | **0.150** | — | — |

### What this means

1. **Webcam**: performance collapses to chance level. The model has no frame of reference for webcam-style compression, sensor noise, and white balance.
2. **Marketplace**: severe **directional bias**. **17 of 20 authentic cards are classified as fake**. The model learned that "iPhone/OnePlus-like captures" = evidence of authenticity, and everything else falls into the "fake" region.

**In production, this model would reject 85% of legitimate authentic cards from real users.**

### Why V2 (previous model) appeared to do better

The V2 model predicted "real" almost unconditionally. On test sets dominated by real cards, this gives high accuracy without any discriminative power.

| Model | Behavior | Marketplace accuracy |
|---|---|---:|
| V2 | Always predicts "real" | 20/20 = 1.00 |
| **V3** | Genuinely discriminates | 3/20 = 0.15 |

**V3 is not worse than V2.** V2's "success" is a degenerate strategy: a classifier that always outputs the majority class has high accuracy but zero utility.

V3's failure is **informative**: it reveals exactly where the learned representation breaks down.

### Why the domain shift is directional

The model treats out-of-distribution **appearance** as evidence of counterfeiting. It has no positive examples of "real" outside the training distribution, so unfamiliar-looking images fall into the "fake" cluster.

**This is not a failure of the architecture.** It is a fundamental limitation of the dataset: two cameras and one capture protocol are insufficient to learn device-invariant authenticity features.

**No amount of architectural sophistication fixes this.** The solution is **more data diversity**, not a better model.

---

## 👁️ Model Interpretability (Grad-CAM)

Grad-CAM reveals what the model actually looks at.

### Key observations

1. **Text regions dominate attention**: card name, attack names, rules text. The model uses **typography, spacing, and print quality** as authenticity signals.
2. **Trainer cards are harder**: all 4 blind-test errors are Trainer cards. Grad-CAM shows attention concentrated on the name and rules text, while the artwork receives little attention.
3. **The model does NOT use the hologram as a shortcut**: 100% accuracy on holographic cards never seen in training.
4. **No language shortcut**: 8/8 Spanish cards correct, with attention on typography, not on linguistic content.
5. **Diffuse attention on domain-shifted errors**: heatmaps spread across the image without clear focus, indicating "guessing" based on low-level unfamiliar features.

### Why this matters

Grad-CAM revealed patterns that accuracy alone could not:

| Observation | Insight |
|---|---|
| Text regions dominate attention | Model relies on typography, which may not transfer across print qualities |
| Trainer cards are harder | Future work should add more Trainer card training data |
| Holograms don't break the model | Model isn't learning trivial material shortcuts |
| Domain-shifted predictions have diffuse attention | Confidence ≠ reliability under shift |

---

## 🧪 Why External Validation Matters

**A high test score is not evidence of real-world generalization.**
Training performance
↓
Internal validation
↓
Held-out test set
↓
Blind test (unseen cards, same protocol) → still high
↓
External test (new expansion, same protocol) → still high
↓
Real-world test (new camera, new conditions) → COLLAPSE


The project treats evaluation as a **progression of difficulty**. Only the last stage reveals what the model actually learned.

---

## 🛠️ Technology Stack

| Component | Version |
|---|---|
| Python | 3.10.14 |
| PyTorch | 2.5.1 |
| Torchvision | 0.20.1 |
| Albumentations | 1.4.22 |
| OpenCV | 4.10.0 |
| scikit-learn | 1.7.2 |
| Pillow | 9.4.0 |
| Pandas | 2.3.3 |
| CUDA | 12.x |
| GPU | NVIDIA RTX 3060 (12 GB) |
| Jupyter Notebook | — |

---

## 📁 Repository Structure

```text
pokemon-card-authenticity/
│
├── README.md
├── LICENSE
├── .gitignore
├── requirements.txt
│
├── notebooks/
│   ├── 01_data_audit.ipynb
│   ├── 02_split_v4.ipynb
│   ├── 03_exif_correction.ipynb
│   ├── 04_training_v3.ipynb
│   ├── 05_internal_test.ipynb
│   ├── 06_blind_test_262.ipynb
│   ├── 07_external_test_47.ipynb
│   └── 08_gradcam_analysis.ipynb
│
├── src/
│   ├── __init__.py
│   ├── dataset.py
│   ├── transforms.py
│   ├── model.py
│   ├── train.py
│   ├── metrics.py
│   ├── gradcam.py
│   └── utils.py
│
├── data/
│   └── README.md          # Not included: photographs of physical cards
│
├── models/
│   └── README.md          # Checkpoint download instructions
│
├── results/
│   ├── internal_test/
│   ├── blind_test_262/
│   ├── external_test_47/
│   ├── real_world_tests/
│   ├── cross_domain/
│   ├── gradcam/
│   └── figures/
│
└── docs/
    ├── 01_dataset.md
    ├── 02_pipeline.md
    ├── 03_training.md
    ├── 04_evaluation.md
    ├── 05_domain_shift.md
    ├── 06_interpretability.md
    ├── 07_limitations.md
    ├── 08_future_work.md
    └── CHANGELOG.md



