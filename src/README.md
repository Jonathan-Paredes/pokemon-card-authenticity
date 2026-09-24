# Source Code

Modularized Python code for training and evaluation.

## Modules

| Module | Purpose |
|---|---|
| `dataset.py` | PyTorch `Dataset` classes (PhoneDataset, InferenceDataset) |
| `transforms.py` | Albumentations pipelines for train/eval |
| `model.py` | EfficientNet-B0 builder |
| `train.py` | Training loop with mixed precision |
| `metrics.py` | Metric computation with per-domain breakdown |
| `gradcam.py` | Grad-CAM helpers |
| `utils.py` | EXIF correction, file utilities |

## Usage

```python
from src.model import build_model
from src.dataset import PhoneDataset
from src.transforms import get_train_transforms, get_eval_transforms
from src.train import train_one_epoch, evaluate

model = build_model(num_classes=2, pretrained=True)
# ...