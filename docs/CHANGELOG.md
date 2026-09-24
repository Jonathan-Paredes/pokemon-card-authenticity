# Changelog

All notable changes to the model, dataset, and pipeline are documented here.

The project has three main versions: V1 (initial), V2 (subset), and V3 (current).

---

## V3 — EfficientNet-B0 (current)

**Released**: 2026-09

### Model
- Backbone: **EfficientNet-B0** (ImageNet pretrained)
- Parameters: 5.3M
- Input size: 224 × 224
- Training: 30 epochs, AdamW, cosine schedule, mixed precision
- Selection metric: mean F1 across iPhone and OnePlus
- Label smoothing: 0.1
- Seed: 42

### Dataset
- 76 physical cards (38 real/fake pairs)
- 2,570 images
- **EXIF-corrected** (all images orientation=1)
- **Card-level split**, no leakage verified 5 ways
- View names normalized to common vocabulary

### Pipeline
- Albumentations-based augmentation
- No vertical flip, no 90° rotations (canonical card layout)
- Consistent normalization between training and evaluation

### Results

| Test | N | Result |
|---|---:|---:|
| Internal | 558 | F1 = 0.978 |
| Blind | 262 | F1 = 0.984 |
| External | 47 | Real recall = 0.957 |
| Cross-domain (iPhone→OnePlus) | — | F1 drop = 0.119 |
| Cross-domain (OnePlus→iPhone) | — | F1 drop = 0.070 |
| Webcam | 29 | Accuracy = 0.552 |
| Marketplace | 20 | Real recall = 0.150 |

### Key findings

1. **Model generalizes to new cards** with the same protocol (blind test: F1 0.984).
2. **Model generalizes to new expansions** with the same protocol (external test: recall 0.957).
3. **Model does NOT generalize to new conditions** (webcam: 0.552, marketplace: 0.150).
4. **Model does not use the hologram as a shortcut** (100% on unseen holographic cards).
5. **Model does not use language as a shortcut** (100% on Spanish cards).
6. **Grad-CAM reveals reliance on text regions** and difficulty with Trainer cards.

### Contributions

- Detection and correction of data leakage from earlier versions.
- Detection and correction of EXIF orientation issue.
- Comprehensive domain shift analysis.
- Card-level split with 5 independent anti-leakage checks.
- Honest documentation of failure modes.

---

## V2 — ResNet18 (subset)

**Released**: earlier (date not recorded)

### Model
- Backbone: ResNet18
- Parameters: 11.2M
- Trained on a subset of the dataset (~38 cards)

### Behavior
The V2 model predicted "real" almost unconditionally.

### Reported results
On internal test with images from the training distribution, V2 reported **99.44% accuracy**.

### Issues discovered later

1. **Data leakage**: image-level split allowed the same physical card to appear in both training and test. The 99.44% accuracy was inflated by this leakage.

2. **Degenerate behavior**: V2's "high accuracy" on real-dominant test sets was a consequence of always predicting the majority class, not of genuine discrimination.

3. **EXIF orientation not corrected**: iPhone images were processed in landscape orientation, adding noise to the training signal.

### Significance

V2 is not a valid baseline. Its "success" is a methodological artifact. The V3 model was built specifically to correct these issues.

### Reported metrics (for historical reference only)

| Metric | Value | Note |
|---|---:|---|
| Internal accuracy | 0.9944 | Inflated by leakage |
| Marketplace accuracy | 20/20 | Degenerate — always predicts "real" |

---

## V1 — Initial ResNet18

**Released**: earliest (date not recorded)

### Model
- Backbone: ResNet18
- Parameters: 11.2M
- Trained on the initial dataset (~110 cards, 1,084 images)

### Reported results
Internal accuracy: 99.44%.

### Issues discovered

Same as V2:
- Data leakage (image-level split)
- EXIF orientation not corrected
- Inflated accuracy

### Significance

V1 is the starting point of the project. The issues discovered during V3 development (leakage, EXIF) apply retroactively to V1's reported numbers.

---

## Version comparison

| Version | Backbone | Cards | Images | Internal F1 | Notes |
|---|---|---:|---:|---:|---|
| V1 | ResNet18 | 110 | 1,084 | 0.994* | Leakage; inflated |
| V2 | ResNet18 | ~38 | ~1,000 | — | Degenerate classifier |
| **V3** | **EfficientNet-B0** | **76** | **2,570** | **0.978** | **Current; verified** |

\* Reported with data leakage; not comparable to V3.

## Migration notes

If you have models or results from V1 or V2:

- **Do not use V1 or V2 metrics** for comparison with V3. They are affected by leakage.
- **Retrain** using the V3 pipeline if you need a ResNet18 baseline.
- **Re-evaluate** on the V3 test split for fair comparison.

## Planned versions

### V3.1 (planned)

Domain-augmented version:
- Stronger augmentation (compression, background replacement)
- Same architecture
- Goal: reduce domain shift gap

### V4 (planned)

Dataset-expanded version:
- 200+ physical cards
- 5+ cameras
- 100+ holographic cards
- Multiple seeds, cross-validated

### V5 (speculative)

Foundation model version:
- DINOv2 or CLIP backbone
- Linear probe or LoRA
- Expected: better cross-domain generalization

## Related documents

- [05_domain_shift.md](05_domain_shift.md) — the domain shift finding
- [07_limitations.md](07_limitations.md) — current limitations
- [08_future_work.md](08_future_work.md) — planned improvements
-
