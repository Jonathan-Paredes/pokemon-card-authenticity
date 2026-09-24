
---

## Documento 4 — `docs/04_evaluation.md`

markdown
# 04 — Evaluation

## Overview

The model was evaluated on **four independent test sets**, each with 
increasing difficulty:

| # | Test | N | Same camera | Same cards | Same expansion |
|---|---|---|---|---|---|
| 1 | Internal | 558 | Yes | No | Yes |
| 2 | Blind | 262 | Yes | No | Yes (mostly) |
| 3 | External | 47 | Yes | No | **No** |
| 4 | Real-world | 49 | **No** | No | No |

Test 4 (real-world) is discussed separately in `05_domain_shift.md`.

## Test 1 — Internal

Standard held-out test set from the original dataset.

| Metric | Value |
|---|---|
| Images | 558 (279 real, 279 fake) |
| Accuracy | 0.9785 |
| F1 | 0.9783 |
| AUC | 0.9811 |
| F1 iPhone | 0.9883 |
| F1 OnePlus | 0.9695 |
| Gap (|iPhone − OnePlus|) | 0.0188 |

## Test 2 — Blind test

262 images from cards that were captured after the initial dataset 
was frozen but **with the same cameras and protocol**.

| Metric | Value |
|---|---|
| Images | 262 (134 real, 128 fake) |
| Accuracy | 0.9847 |
| F1 | 0.9841 |
| AUC | 0.9998 |
| Errors | **4 / 262 (1.5%)** |

### Error analysis

The 4 errors share a striking pattern:

| File | Label | Pred | Conf | Type |
|---|---|---|---|---|
| C032 | fake | real | 0.769 | Trainer (Sacred Ash) |
| C033 | fake | real | 0.675 | Trainer (Ethan's Adventure) |
| C026 | fake | real | 0.657 | Trainer (Team Rocket's Venture Bomb) |
| C029 | fake | real | 0.631 | Trainer (Team Rocket's Petrel) |

**All four are Trainer cards** (not Pokémon). Trainer cards have:

- Large plain backgrounds
- Simpler graphics
- Text-dominant layouts

This suggests the model relies partially on **text and layout** 
features that are less discriminative in Trainer cards. Detailed in 
`06_interpretability.md`.

### Cross-camera consistency

Since each card was captured with both cameras, we can check 
prediction consistency at the card level:

| Category | Count |
|---|---|
| Both cameras correct | 127 |
| Only OnePlus correct | 3 |
| Only iPhone correct | 1 |
| Both cameras wrong | **0** |

Zero cases of "both wrong". This indicates the model's errors are 
**borderline**, not systematic per card.

## Test 3 — External (new expansion)

47 images of cards from an **expansion the model has never seen**, 
captured with the same cameras but with **flash** and partial 
framing. All 47 are **authentic**.

| Metric | Value |
|---|---|
| Images | 47 (all real) |
| Predicted real | 45 |
| Predicted fake | 2 (false positives) |
| Recall (real) | **0.9574** |
| Mean confidence | 0.904 |
| Median confidence | 0.927 |

### The 2 false positives

| File | P(fake) | Card type |
|---|---|---|
| C037 | 0.80 | Trainer (Sacred Charm) |
| C044 | 0.44 | Pokémon EX (Mega Gengar) |

C037 is again a Trainer card with white background. C044 is a 
holographic Pokémon EX card, where the model's confidence was near 
the decision boundary.

## Test 4 — Real-world conditions

See `05_domain_shift.md` for the detailed analysis. Summary:

| Source | N | Recall (real) |
|---|---|---|
| Webcam images | 29 | 0.545 |
| Marketplace listing photos | 20 | **0.150** |

Both indicate severe domain shift when conditions differ from the 
training protocol.
