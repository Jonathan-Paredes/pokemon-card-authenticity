# 06 — Model Interpretability (Grad-CAM)

## Motivation

High accuracy alone does not tell us *how* the model makes predictions. To understand what visual features the model relies on, **Grad-CAM** was applied to both correct and incorrect predictions.

This matters because:

1. A model could rely on **shortcuts** that don't generalize.
2. Failure analysis requires knowing **what the model attended to**.
3. Trust in deployment requires understanding **why predictions are made**.

## Method

Grad-CAM computes a heatmap of gradient-weighted activations in the last convolutional layer, highlighting image regions that most strongly influenced the prediction.

**Target layer**: `model.features[-1]` (last block of EfficientNet-B0).

**Library**: `pytorch-grad-cam`.

**Visualization**: heatmaps overlaid on the input image, with warmer colors indicating higher attention.

## Key findings

### 1. The model attends to text regions

In most correctly classified images, the heatmap concentrates on:

- **Card name** (top of card)
- **Attack names and descriptions**
- **Trainer card text blocks**

This suggests the model uses **typography, spacing, and printing quality of text** as authenticity signals.

**Implication**: the model has learned that counterfeit cards often have subtle printing differences in text regions — which is a real-world authenticator strategy.

### 2. Trainer cards are harder

The 4 errors in the blind test (C032, C033, C026, C029) were all **Trainer cards**. Grad-CAM shows attention concentrated on the **name region** and the **rules text block**, while the artwork receives little attention.

Trainer cards have:

- Large uniform backgrounds
- Simpler illustrations
- Text-dominated layouts

This reduces the amount of visual signal compared to Pokémon cards, where artwork provides additional texture patterns the model can leverage.

**Implication**: future work should add more Trainer card training data.

### 3. The model does NOT use the hologram as a shortcut

Despite holographic cards being **entirely absent from training**, authentic holographic cards in the blind test were classified correctly **30/30 (100%)**.

Grad-CAM confirms attention is distributed across the card, not concentrated on the holo region.

**Implication**: the model learned features that transfer to holographic cards without ever seeing them during training. This is a strong positive result.

### 4. No language shortcut

The external test included 8 cards in Spanish. All 8 were correctly classified (8/8). Grad-CAM attention is on typography and printing quality, not on the specific language content.

**Implication**: the model has not learned to associate language with authenticity. It uses visual features that are language-independent.

### 5. Diffuse attention on domain-shifted errors

Grad-CAM on the marketplace false positives (see [05_domain_shift.md](05_domain_shift.md)) shows attention spread across the entire card without a clear focus region.

**Implication**: the model is "guessing" based on unfamiliar low-level features, not identifying reliable authenticity indicators. This is consistent with the domain shift hypothesis.

## Attention patterns summary

| Scenario | Attention concentration | Interpretation |
|---|---|---|
| Correct Pokémon card | Text regions + artwork | Reliable features |
| Correct Trainer card | Text + rules block | Text-driven |
| Trainer card error | Rules text block only | Insufficient artwork signal |
| Holographic (unseen) | Distributed across card | Not using holo as shortcut |
| Marketplace error | Diffuse, no focus | Guessing under domain shift |

## Visual evidence

Grad-CAM figures are stored in `results/gradcam/`:

- `blind_test_errors.png` — the 4 blind test errors with heatmaps
- `blind_test_successes.png` — high-confidence correct predictions
- `external_test_errors.png` — the 2 external test false positives
- `marketplace_errors.png` — the marketplace domain shift failures
- `mean_attention_success_vs_error.png` — average heatmap for correct vs incorrect predictions

## Grad-CAM as a diagnostic tool

Grad-CAM revealed patterns that accuracy alone could not:

| Observation | Insight |
|---|---|
| Text regions dominate attention | Model relies on typography, which may not transfer to other print qualities |
| Trainer cards are harder | Future work should add more Trainer card training data |
| Holograms don't break the model | The model isn't learning trivial material shortcuts |
| Domain-shifted predictions have diffuse attention | Confidence ≠ reliability under shift |
| No language-specific attention | Model uses visual, not textual, features |

## What we did NOT find

Some hypotheses that Grad-CAM **did not support**:

1. **No watermark fixation**: we suspected the model might use the Pokémon watermark as a shortcut. It doesn't — attention on watermarks is low.

2. **No border fixation**: we suspected edge detection might dominate. It doesn't — borders receive moderate attention.

3. **No background fixation in-domain**: we suspected the model might use training backgrounds as a signal. It doesn't in-domain — attention is on the card. (It may in the marketplace failures, but we can't confirm.)

These negative results are useful: they rule out several shortcut hypotheses.

## Limitations of Grad-CAM

Grad-CAM is an interpretability tool, not definitive evidence of what the model "understands".

Limitations:

1. **Coarse resolution**: heatmaps are low-resolution (typically 7×7 for EfficientNet-B0's last layer).
2. **Not causally validated**: attention does not imply causation.
3. **Layer-dependent**: results depend on which layer is targeted.
4. **Ambiguity**: multiple hypotheses can explain the same heatmap.

Grad-CAM should be interpreted alongside other evidence (accuracy metrics, error analysis, ablation studies).

## Related documents

- [04_evaluation.md](04_evaluation.md) — evaluation methodology
- [05_domain_shift.md](05_domain_shift.md) — domain shift analysis with Grad-CAM evidence
- [07_limitations.md](07_limitations.md) — limitations of the model
