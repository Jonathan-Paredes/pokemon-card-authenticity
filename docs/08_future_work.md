# 08 — Future Work

This document outlines planned improvements and extensions of the project.

## Dataset expansion (highest priority)

The single most important improvement is **dataset diversity**. The current model is strong within its training distribution but fails outside it (see [05_domain_shift.md](05_domain_shift.md)).

### 1. Add more cameras

| Camera type | Rationale |
|---|---|
| Modern Android (Samsung, Xiaomi, Pixel) | Different color science and processing |
| Older smartphones | Broader sensor distribution |
| Webcam | Simulate desktop use case |
| Flatbed scanner | Simulate seller listing style |
| DSLR / mirrorless | High-quality reference captures |

**Target**: at least 4-5 distinct sources, each contributing 100+ images.

### 2. Add holographic cards

Currently excluded (see [07_limitations.md](07_limitations.md)). Target: 50+ real + 50+ fake holographic cards.

**Acquisition strategy**:

- Prioritize **LP/MP condition** cards (3-10× cheaper than NM).
- Mixed condition diversity reduces bias toward perfect-condition prints.
- Pair real/fake whenever possible to maintain the paired dataset structure.
- Consider bulk lots to reduce per-card cost.

**Timeline**: coordinated with market conditions (wait for price drops).

### 3. Add marketplace-style images

Directly collect listing photos from public marketplaces with permission.

**Target**: 100+ images representing variable backgrounds, lighting, and compression.

### 4. Add more counterfeit sources

Current counterfeits come from limited vendors. Expanded sources would increase the diversity of counterfeit production methods.

### 5. Multi-angle captures

For each card, capture 4-6 angles to enable angle-invariant training and support holographic analysis.

**Focus on holographic cards** where angle dependence is strongest.

## Model improvements

### 6. Domain-adversarial training (DANN)

Add a domain classifier with gradient reversal layer to force the feature extractor to be camera-invariant.

**Expected benefit**: reduce cross-camera gap and improve generalization to new cameras.

### 7. Foundation models

Replace EfficientNet-B0 with DINOv2, CLIP, or SigLIP backbones.

**Approaches**:

- Frozen backbone + linear probe
- LoRA fine-tuning
- Full fine-tuning (if compute allows)

**Hypothesis**: semantic features from large-scale pretraining transfer better across domains than texture features learned from scratch.

### 8. Multi-task learning

Add auxiliary heads for:

- **Card type**: Pokémon / Trainer / Energy
- **Holographic presence**: no / reverse / full
- **Language**: EN / ES / JP

**Hypothesis**: auxiliary tasks force the model to learn richer representations and prevent text-only shortcuts.

### 9. Ensemble and test-time augmentation

Combine multiple seeds (e.g., 42, 123, 7) and apply TTA at inference time.

**Expected benefit**: 1-2% accuracy improvement and reduced variance.

### 10. Confidence calibration

Apply temperature scaling or Platt scaling to align confidence with empirical accuracy.

**Expected benefit**: better-behaved confidence scores for deployment.

### 11. Uncertainty quantification

Add Monte Carlo Dropout or Deep Ensembles for epistemic uncertainty estimation.

**Expected benefit**: identify when the model is uncertain and should defer to a human.

## Evaluation improvements

### 12. Cross-validated results

Run 3-5 seeds with k-fold card-level CV for robust variance estimates.

**Expected benefit**: tighter confidence intervals, more defensible numbers.

### 13. Adversarial evaluation

Test robustness against:

- Blurred images
- Heavy JPEG compression (quality 20-50)
- Grayscale conversion
- Cropped partial cards
- Rotated / flipped images
- Image with added noise

### 14. Confounder analysis

Explicitly measure the model's dependence on:

- Text typography
- Background color
- Image compression quality
- Card aspect ratio
- Lighting direction

**Method**: systematically vary each factor while holding others constant.

### 15. Statistical significance testing

Formal tests comparing models (V2 vs V3) and conditions (in-domain vs cross-domain).

## Interpretability extensions

### 16. Attribution beyond Grad-CAM

Use:

- **Score-CAM**: cleaner visualizations
- **Integrated Gradients**: pixel-level attribution
- **SHAP**: feature-level attribution

### 17. Attention analysis at view level

Compare Grad-CAM heatmaps across different views (FRONT, WATERMARK, etc.) to understand which views are most informative.

### 18. Human study

Show Grad-CAM maps to Pokémon collectors and card graders. Compare model attention with human expert attention.

**Question**: does the model look at the same regions experts do?

## Deployment considerations

### 19. Mobile deployment

Convert the model to ONNX or TFLite for on-device inference.

**Target devices**: modern Android and iOS smartphones.

### 20. Human-in-the-loop workflow

Design a workflow where:

- High-confidence predictions (>0.95) are auto-accepted
- Medium-confidence predictions (0.7-0.95) are flagged for review
- Low-confidence predictions (<0.7) require manual inspection

### 21. Model card

Prepare a "Model Card" documenting:

- Intended use
- Limitations
- Known failure modes
- Ethical considerations

### 22. Web demo

A simple Gradio or Streamlit app allowing users to upload a card photo and receive a prediction with confidence and (optionally) Grad-CAM.

**Value**: demonstrates the system to non-technical users.

## Publication roadmap

The current results are sufficient for:

### Short paper (workshop-level)

**Title**: "Data Leakage and EXIF Orientation in Photographic Card Authentication: A Case Study"

**Focus**: methodological pitfalls discovered during the project.

### Medium paper (conference-level)

**Title**: "Domain Shift in Photographic Counterfeit Detection: When 98% Accuracy Means 15% Recall"

**Focus**: the domain shift analysis and its methodological implications.

### Full paper (journal-level)

Would benefit from:

- Dataset expansion (200+ cards, 5+ cameras)
- Holographic examples (100+)
- Multiple architecture comparisons
- Human study

## What NOT to prioritize

Things that seem interesting but are **not** high-value:

1. **Fancy architectures** (ViT, EfficientNetV2) without more data: the bottleneck is data diversity, not model capacity.
2. **Adversarial robustness** against deliberate attacks: not the real threat model.
3. **Extreme preprocessing** (super-resolution, denoising): may introduce artifacts.
4. **Multi-modal inputs** (text descriptions, metadata): not available in practice.

## Related documents

- [05_domain_shift.md](05_domain_shift.md) — the problem to solve
- [07_limitations.md](07_limitations.md) — the limitations to address
