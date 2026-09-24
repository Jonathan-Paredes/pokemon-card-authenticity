# 05 — Domain Shift Analysis (Critical Finding)

## Motivation

The model achieved strong results on internal and blind tests (F1 ≥ 0.978). However, all these tests shared the same acquisition protocol as training:

- Same two smartphones (iPhone, OnePlus)
- Same lighting conditions
- Same backgrounds
- Same framing methodology

**The critical question**: does the model generalize to images captured outside this protocol?

This document answers that question, and the answer is **no**.

## The experiment

Two additional test sets were collected under realistic conditions:

### Test 4a — Webcam images

29 images captured with a laptop webcam, with a mix of authentic (n=11) and counterfeit (n=18) cards.

**None of these conditions were present during training:**
- Different sensor (webcam vs smartphone)
- Lower resolution
- Different noise patterns
- Aggressive automatic white balance
- Unknown compression pipeline

### Test 4b — Marketplace listing photos

20 images of authentic Pokémon cards taken from online marketplace listings (MercadoLibre, eBay-style listings).

These represent the "seller photo" distribution:
- Variable backgrounds (tables, fabrics, plain surfaces)
- Uncontrolled lighting (natural, ceiling, flash)
- Platform-specific compression (WhatsApp, marketplace uploads)
- Unknown framing

All 20 are authentic — the reference use case is verifying a seller's photo of a card they claim is genuine.

## Results

| Test | N | Real recall | Fake recall | Overall |
|---|---:|---:|---:|---:|
| Blind (control) | 262 | 0.988 | 0.977 | 0.984 |
| External | 47 | 0.957 | — | — |
| **Webcam** | 29 | **0.545** | 0.556 | **0.552** |
| **Marketplace** | 20 | **0.150** | — | — |

### Reading the numbers

**Webcam**: accuracy drops to chance level (~0.55). The model treats webcam-specific features (low resolution, different sensor noise, aggressive white balance) as noise.

**Marketplace**: severe directional bias. **17 of 20 authentic cards are classified as fake**. This is not random — it is a systematic failure mode.

## Why the failure is directional

The marketplace errors are **all false positives** (real → fake), not false negatives.

This asymmetry is significant. It suggests the model learned:

