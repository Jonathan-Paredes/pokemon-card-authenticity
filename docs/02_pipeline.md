# 02 — Preprocessing Pipeline

## Overview

The preprocessing pipeline consists of five stages:




## [1] EXIF orientation correction

### Problem

Smartphone cameras store images in **landscape orientation** in pixel 
space, and add an **EXIF Orientation tag** (value `6` or `8`) that 
tells the viewer to rotate the image for display.

- iPhone: `Orientation = 6` for portrait captures
- OnePlus: images saved with corrected pixels, `Orientation = 1`

When loaded with `cv2.imread()` (which ignores EXIF), iPhone images 
appear rotated 90°. If the model is trained with this inconsistency, 
it learns spurious orientation features that don't generalize.

### Solution

Every image is passed through `PIL.ImageOps.exif_transpose()`, which:

1. Reads the EXIF Orientation tag.
2. Physically rotates the pixels to match.
3. Removes the tag so downstream tools don't double-rotate.

The corrected images are saved to `Dataset_imagenes_fixed/` and used 
for all training and evaluation.

### Verification

| Stage | iPhone | OnePlus |
|---|---|---|
| Before | 589 vertical + EXIF=6 (horizontal pixels) | 699 vertical + EXIF=0 |
| After | 589 vertical + EXIF=1 | 699 vertical + EXIF=1 |

All 2570 images end up with consistent orientation.

## [2] Card-level split

### Why card-level and not image-level

Multiple images can belong to the same physical card (different views, 
different cameras). If we split by image, the same card could appear 
in both train and test:


This is a **data leakage** scenario: the model can memorize card-
specific features (artwork, text, printing quirks) and appear to 
generalize when it doesn't.

### Split procedure

Groups are defined by `card_id`. Using `GroupShuffleSplit` from 
scikit-learn:

| Split | Cards | Images |
|---|---|---|
| train | 48 | 1554 |
| validation | 12 | 372 |
| test | 16 | 644 |
| **Total** | **76** | **2570** |

Every card appears in exactly one split. Real/fake pairs remain 
paired: `C001_real` and `C001_fake` always share a split.

## [3] Anti-leakage verification

Five independent checks were performed:

| Check | Result |
|---|---|
| `card_id` in >1 split | **0** |
| `filename` in >1 split | **0** |
| `physical_id` in >1 split | **0** |
| Identical `md5` across splits | **0** |
| Near-duplicates (32×32 perceptual hash) across splits | **0** |

All checks pass. The split is leakage-free.

## [4] View name normalization

iPhone and OnePlus use different view naming conventions:

| iPhone | OnePlus | Normalized |
|---|---|---|
| `FRONT_IPHONE` | `FRONT_ONEPLUS` | `FRONT` |
| `FRONT_IPHONE_ABOVE` | `FRONT_ONEPLUS_ABOVE` | `FRONT_ABOVE` |
| `FRONT_IPHONE_CHARACTERISTICS` | `FRONT_ONEPLUS_CHARACTERISTICS` | `CHARACTERISTICS` |
| ... | ... | ... |

Without this normalization, if views were used as auxiliary features, 
the model could trivially distinguish cameras by view name alone.

## [5] Online augmentation

Augmentations are applied during training with Albumentations. Key 
choices and their rationale:

| Transform | Rationale |
|---|---|
| `RandomResizedCrop` (0.7–1.0) | Simulate different framings |
| `HorizontalFlip` | Cards are symmetric in this context |
| `Rotate(±12°)` | Small rotation tolerance (not 90°) |
| `ColorJitter(0.35, 0.35, 0.35, 0.08)` | Camera-style color variation |
| `RandomGamma(75–125)` | Exposure variation |
| `GaussNoise(5–30)` | Sensor noise |
| `MotionBlur(blur_limit=3)` | Hand-shake |
| `ISONoise` | Low-light sensor artifacts |

**Not used**: `VerticalFlip`, `RandomRotate90`, `Transpose`. Rationale: 
the watermark, text, and card layout have canonical orientation. 
Flipping vertically would introduce invariances that don't hold in 
the real world.

## Consistency between training and evaluation

Both training and evaluation use the same **normalization**:

```python
mean = (0.485, 0.456, 0.406)  # ImageNet
std  = (0.229, 0.224, 0.225)



---

## Documento 3 — `docs/03_training.md`

```markdown
# 03 — Training

## Model

**EfficientNet-B0** with ImageNet-pretrained weights, fine-tuned 
end-to-end.

| Parameter | Value |
|---|---|
| Architecture | EfficientNet-B0 |
| Pretrained | ImageNet-1k (V1) |
| Total parameters | ~5.3M |
| Input size | 224 × 224 × 3 |
| Output classes | 2 (real, fake) |
| Modified layer | `classifier[1]`: Linear(1280 → 2) |

## Training configuration

| Parameter | Value |
|---|---|
| Optimizer | AdamW |
| Learning rate | 1e-4 |
| Weight decay | 1e-4 |
| Scheduler | CosineAnnealingLR (T_max=30) |
| Loss | CrossEntropyLoss with label_smoothing=0.1 |
| Batch size | 64 |
| Max epochs | 30 |
| Early stopping | patience=7 |
| Mixed precision | Yes (torch.cuda.amp) |
| Seed | 42 |

### Why label smoothing

With only 76 physical cards, memorization is a real risk. Label 
smoothing (ε=0.1) prevents the model from becoming overconfident 
on training samples and improves calibration.

### Why mixed precision

RTX 3060 (12 GB) with fp16 autocast allows batch 64 comfortably and 
speeds up training by ~1.5×.

## Model selection criterion

The best checkpoint was selected by **cross-domain F1**:

```python
score = (F1_iphone + F1_oneplus) / 2





