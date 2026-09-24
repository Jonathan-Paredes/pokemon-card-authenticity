# 01 — Dataset Construction

## Overview

The dataset consists of photographs of **physical Pokémon cards**, each labeled as either authentic (`real`) or counterfeit (`fake`).

The design of the dataset is a deliberate trade-off:

- **On one hand**, the acquisition protocol was kept consistent to enable clean training and clean evaluation.
- **On the other hand**, this consistency introduces **domain bias**: the model only ever sees two camera setups.

This tension between consistency and diversity is a central theme of the project.

## Composition

| Metric | Value |
|---|---:|
| Physical cards | 76 |
| Real/Fake pairs | 38 |
| Total images | 2,570 |
| Cameras | 2 (iPhone, OnePlus) |
| Views per card | 7–10 |
| Languages | English, Spanish |

## Physical card structure

Each **physical card** has a unique identifier (`card_id`) and is photographed multiple times. Two key properties:

### 1. Real/fake pairing

For each authentic card `C0XX`, a counterfeit card `C0XX` of the same design was acquired. These are **two different physical objects**, not two photos of the same object.

This pairing is important because it enables:

- Balanced real/fake training.
- Direct comparison between authentic and counterfeit versions of the same design.
- Card-level splits that keep both versions in the same split.

### 2. Multiple photographs per card

Each card is captured from multiple angles and regions. This introduces a critical constraint: **splitting must be at the card level, not the image level** (see [02_pipeline.md](02_pipeline.md)).

## Capture protocol

Each card was photographed with **two different smartphones**:

| Device | Sensor | Typical resolution |
|---|---|---|
| iPhone | Apple iOS camera | 4032 × 3024 (landscape pixels, EXIF=6) |
| OnePlus | Android camera | 4000 × 3000 (portrait pixels, EXIF=1) |

### Views captured

| View | Description |
|---|---|
| `FRONT` | Full frontal view |
| `FRONT_ABOVE` / `FRONT_BELOW` | Slight angle variations |
| `FLASH` | Captured with flash |
| `LOWER_LEFT` / `UPPER_LEFT` | Corner crops |
| `WATERMARK` | Watermark region close-up |
| `CHARACTERISTICS` | Card-specific visual feature |
| `ANGLE15` / `ANGLE45` / `ANGLE75` | OnePlus only — angled captures |

Different views target different authenticity-relevant features:

- **Frontal views**: overall layout, colors, typography.
- **Corner crops**: edge cut quality, border precision.
- **Watermark views**: the watermark region (a common counterfeit weak point).
- **Flash views**: highlight differences in print finish and gloss.
- **Angled views**: reveal texture and holographic behavior.

## Class balance

| Class | Images | Physical cards |
|---|---:|---:|
| real | 1,282 | 38 |
| fake | 1,288 | 38 |
| **Total** | **2,570** | **76** |

The dataset is **class-balanced at the physical card level**. This is intentional: it prevents the model from learning a trivial majority-class prior.

Balanced class distribution means:

- A classifier that always predicts "real" gets 50% accuracy.
- Any performance above 50% is genuine signal.

## Known acquisition biases

The dataset has inherent limitations. These are **known, documented, and analyzed**:

### 1. Two cameras only

All images come from an iPhone and a OnePlus device. The model has never seen:

- Webcams (low resolution, different color science)
- DSLRs / mirrorless cameras (large sensors, shallow depth of field)
- Flatbed scanners (even lighting, no perspective)
- Other smartphones (Samsung, Xiaomi, Pixel — different sensors and processing)

**Impact**: severe performance drop on non-training cameras (see [05_domain_shift.md](05_domain_shift.md)).

### 2. Controlled capture conditions

Backgrounds, lighting, and framing were consistent during acquisition.

**Impact**: the model may use background or lighting cues as shortcuts for classification. When these cues change, performance collapses.

### 3. Limited counterfeit diversity

Counterfeit cards in the dataset come from specific sources. Different manufacturers, vintages, or quality levels are not represented.

**Impact**: unknown performance on novel counterfeit production methods.

### 4. Holographic cards excluded

7 pairs of holographic cards were excluded from training due to insufficient sample size (see [07_limitations.md](07_limitations.md)).

**Impact**: the model has not been trained on the holographic subpopulation. Performance on authentic holographic cards was evaluated separately.

### 5. Single-angle captures per view

Even though multiple views were captured, each view is a single frontal angle. Holographic cards in particular are strongly angle-dependent.

**Impact**: holographic authentication from a single image is unreliable.

## What is NOT in the dataset

- Holographic cards (excluded from training; used only as a test case).
- Cards from the newest expansions (planned for V4).
- Images from cameras other than iPhone/OnePlus (this is the domain shift problem).
- Images with non-controlled backgrounds (except in real-world tests).
- Videos or multi-angle sequences.

## Dataset evolution

| Version | Cards | Images | Notes |
|---|---:|---:|---|
| V1 | 110 | 1,084 | Initial collection (see [CHANGELOG.md](CHANGELOG.md)) |
| V2 | ~38 pairs | ~1,000 | Subset with fewer views |
| **V3** | **76** | **2,570** | Current: cleaned, EXIF-corrected, view-normalized |

V3 is the canonical version used in all current experiments.

## Acquisition cost and practical notes

For readers considering reproducing this dataset, some practical notes:

- **Card acquisition is the main cost.** Authentic Pokémon cards (especially holographic) are expensive in the collector market. Counterfeits are cheap but require careful sourcing.
- **The realistic recommendation** is to prioritize **LP/MP condition cards** (lightly/moderately played) over Near-Mint:
  - 3–10× cheaper.
  - Better represent real-world use cases (users are authenticating worn cards, not only pristine ones).
  - Reduce bias toward perfect-condition prints.
- **Storage**: 76 cards fit in a small binder. Scaling to hundreds requires dedicated storage.

## Related documents

- [02_pipeline.md](02_pipeline.md) — how the raw photographs are processed
- [05_domain_shift.md](05_domain_shift.md) — why the dataset design causes real-world failure
- [07_limitations.md](07_limitations.md) — detailed limitations
- [08_future_work.md](08_future_work.md) — expansion plans
