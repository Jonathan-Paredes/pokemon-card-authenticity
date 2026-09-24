# 07 — Limitations

This document lists limitations that are **known, acknowledged, and documented**. They are not hidden from readers; they are part of the project's contribution to understanding when and how the model fails.

## Dataset limitations

### 1. Two cameras only

All training data comes from an iPhone and a OnePlus device. The model has not learned to be invariant to camera characteristics (sensor noise, color science, sharpening, compression).

**Impact**: severe performance drop on webcam and marketplace images (see [05_domain_shift.md](05_domain_shift.md)).

### 2. Controlled capture conditions

Backgrounds, lighting, and framing were consistent during acquisition.

**Impact**: the model may use background or lighting cues as shortcuts. When these cues change, performance collapses.

### 3. Limited counterfeit diversity

Counterfeit cards in the dataset come from specific sources. The model has not seen counterfeits from different manufacturers, vintages, or quality levels.

**Impact**: unknown performance on novel counterfeit production methods.

### 4. Holographic cards excluded

7 pairs of holographic cards were excluded from training due to insufficient sample size.

**Rationale**: with only 14 base images (7 real + 7 fake), the model would memorize rather than learn holographic features.

**Impact**: the model has not been trained on the holographic subpopulation. Performance on authentic holographic cards is verified in the blind test (100% correct), but counterfeit holographic cards are untested.

**Note**: expanding to 50+ holographic pairs is planned in [08_future_work.md](08_future_work.md).

### 5. No multi-angle captures

Holographic cards and other reflective features depend heavily on viewing angle. The current protocol uses a single frontal angle per view.

**Impact**: holographic classification may not be reliable from single-view images.

### 6. Limited language diversity

Only 8 Spanish cards are in the test set. English dominates.

**Impact**: strong performance on Spanish is observed, but statistical confidence is limited by sample size.

### 7. Small dataset at the card level

76 physical cards is small compared to large-scale vision datasets (thousands to millions).

**Impact**: limited statistical power in evaluation, higher variance across random seeds, risk of subtle biases.

## Model limitations

### 8. Confidence is not reliability

As shown in [05_domain_shift.md](05_domain_shift.md), the model can be highly confident while wrong (e.g., marketplace images at 88% confidence classified as fake but actually real).

**Impact**: model confidence should not be used as a production gate without domain-specific calibration.

### 9. Binary output only

The model outputs only `real`/`fake`. It does not provide:

- Reason for the classification
- Region-of-interest annotations
- Confidence intervals
- Explanation of failure mode

**Impact**: end users cannot audit individual predictions.

### 10. Class balance assumption

Training uses balanced real/fake data. Real-world traffic may have different priors (e.g., mostly authentic). The model's threshold may need re-calibration.

**Impact**: the model's default 0.5 threshold may not be optimal for deployment.

### 11. No calibration

The model outputs softmax probabilities that are not temperature-scaled. Reliability diagrams would likely show miscalibration.

**Impact**: the reported "confidence" is not a well-calibrated probability.

### 12. No uncertainty quantification

The model does not provide confidence intervals or epistemic uncertainty estimates.

**Impact**: on out-of-distribution inputs, the model may output overconfident predictions (as observed in marketplace tests).

## Evaluation limitations

### 13. Small external sample sizes

- Blind test: 262 images
- External test: 47 images
- Real-world tests: 49 images total

Confidence intervals on these are wide. Statistical claims about domain shift effects should be interpreted with caution.

### 14. No cross-validated results

Results are from a single training run with seed 42. Multiple seeds would give a more robust estimate of variance.

### 15. All external cards are authentic

The external 47-image test contains only real cards. It measures false-positive rate but not false-negative rate under domain shift.

### 16. No adversarial evaluation

The model has not been tested against:

- Deliberately blurred images
- Heavy JPEG compression
- Rotation/flip transformations
- Deliberate adversarial perturbations

**Impact**: robustness to image transformations is unknown.

### 17. No confounder analysis

The model's dependence on individual factors (typography, background, compression, lighting) has not been isolated.

**Impact**: we cannot quantify how much each factor contributes to performance.

## Deployment limitations

### 18. Not production-ready

The model is a research prototype. It is not suitable for actual authentication without:

- Expanded training data
- Domain adaptation
- Human-in-the-loop verification
- Legal and business considerations

### 19. No physical feature analysis

Authenticity in the real world involves physical characteristics that cannot be captured from a single photograph:

- Printing texture (raised ink, embossing)
- Paper weight and thickness
- Holographic refraction under specific light
- Edge cut quality
- Odor (yes, this is a real technique used by authenticators)

**Impact**: the model is at most a **first-pass filter**, not a final decision system.

### 20. No mobile/inference optimization

The model has not been converted to ONNX/TFLite or optimized for mobile inference.

**Impact**: deployment on a phone is possible but would require additional engineering.

## Methodological caveats

### 21. Findings are dataset-specific

The specific accuracy numbers and domain shift magnitudes are tied to this particular dataset. Other datasets with different characteristics may show different patterns.

However, the **qualitative findings** (leakage detection, EXIF importance, domain shift in real-world conditions) are general and apply to any image classification project with similar characteristics.

### 22. No formal statistical testing

We have not performed statistical significance tests comparing V2 and V3, or comparing cross-domain results.

### 23. No ablation study

We have not systematically removed individual components (EXIF correction, card-level split, augmentation, etc.) to quantify their contribution.

## Known unknowns

Things we **know we don't know**:

1. How the model behaves on cards from the 1990s-2000s (mostly vintage, different print styles).
2. How it behaves on modern Japanese cards with different layouts.
3. How it behaves on non-Pokémon TCG (Magic, Yu-Gi-Oh).
4. How it behaves on extremely high-quality counterfeits designed to fool experts.
5. How it behaves on cards with intentional physical damage.
6. How it behaves on scans (flatbed) vs photographs.

## Related documents

- [05_domain_shift.md](05_domain_shift.md) — the main limitation in practice
- [08_future_work.md](08_future_work.md) — how these limitations could be addressed
