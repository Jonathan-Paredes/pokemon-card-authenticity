# Notebooks

Ordered walkthrough of the project. Notebooks should be run in order.

| # | Notebook | Purpose |
|---|---|---|
| 01 | `01_data_audit.ipynb` | Explore raw dataset, detect issues |
| 02 | `02_split_v4.ipynb` | Regenerate card-level split with anti-leakage checks |
| 03 | `03_exif_correction.ipynb` | Fix EXIF orientation, save corrected images |
| 04 | `04_training_v3.ipynb` | Train EfficientNet-B0 |
| 05 | `05_internal_test.ipynb` | Evaluate on internal test set |
| 06 | `06_blind_test_262.ipynb` | Blind test evaluation |
| 07 | `07_external_test_47.ipynb` | External test (new expansion) |
| 08 | `08_gradcam_analysis.ipynb` | Grad-CAM visualization and analysis |

## Environment

All notebooks assume:

- Dataset images stored outside the repo (see `../data/README.md`)
- Checkpoints stored outside the repo (see `../models/README.md`)
- Environment variables set per `../README.md`

## Execution time

| Notebook | Approximate time |
|---|---|
| 01 | 1 min |
| 02 | 30 sec |
| 03 | 5 min (processes 2,570 images) |
| 04 | 15 min (30 epochs on RTX 3060) |
| 05-07 | 1 min each |
| 08 | 5 min (Grad-CAM inference) |

Total: ~30 minutes end-to-end.