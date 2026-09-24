# 📚 Documentation Index

This folder contains the full technical documentation of the Pokémon Card Authenticity Detection project.

The documentation is organized as a sequence of self-contained documents, each covering one aspect of the pipeline. Together they form a complete record of the methodology, results, and findings.

## Documents

| # | Document | Description |
|---|---|---|
| 01 | [01_dataset.md](01_dataset.md) | Dataset construction, physical cards, capture protocol |
| 02 | [02_pipeline.md](02_pipeline.md) | Preprocessing, EXIF correction, split, anti-leakage verification |
| 03 | [03_training.md](03_training.md) | Model architecture, hyperparameters, training procedure |
| 04 | [04_evaluation.md](04_evaluation.md) | Internal, blind, external, and real-world tests |
| 05 | [05_domain_shift.md](05_domain_shift.md) | ⭐ Critical finding: performance collapse under real-world conditions |
| 06 | [06_interpretability.md](06_interpretability.md) | Grad-CAM analysis and model attention patterns |
| 07 | [07_limitations.md](07_limitations.md) | Known limitations and their implications |
| 08 | [08_future_work.md](08_future_work.md) | Planned experiments and dataset expansion |
| — | [CHANGELOG.md](CHANGELOG.md) | Version history (V1 → V2 → V3) |

## Quick summary

| Dimension | Value |
|---|---|
| Physical cards | 76 (38 real/fake pairs) |
| Total images | 2,570 |
| Cameras | iPhone + OnePlus |
| Model | EfficientNet-B0 (5.3M params) |
| Internal test F1 | 0.978 |
| Blind test F1 (262 unseen) | 0.984 |
| External test recall (47 new) | 0.957 |
| Real-world performance | **Collapses** — see [05_domain_shift.md](05_domain_shift.md) |

## How to read this documentation

If you only have 5 minutes:

1. Read the **TL;DR** in the root [README.md](../README.md).
2. Read [05_domain_shift.md](05_domain_shift.md) — the central finding.

If you want to reproduce the work:

1. [01_dataset.md](01_dataset.md) — understand the data
2. [02_pipeline.md](02_pipeline.md) — reproduce the preprocessing
3. [03_training.md](03_training.md) — reproduce the training
4. [04_evaluation.md](04_evaluation.md) — understand the evaluation methodology

If you're interested in methodology lessons:

1. [05_domain_shift.md](05_domain_shift.md) — domain shift
2. [06_interpretability.md](06_interpretability.md) — Grad-CAM
3. [07_limitations.md](07_limitations.md) — limitations
