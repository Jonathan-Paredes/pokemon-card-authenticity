
---

## 📄 `docs/03_training.md`

```markdown
# 03 — Training

## Model

**EfficientNet-B0** with ImageNet-pretrained weights, fine-tuned end-to-end.

| Parameter | Value |
|---|---|
| Architecture | EfficientNet-B0 |
| Pretrained | ImageNet-1k (V1) |
| Total parameters | ~5.3M |
| Input size | 224 × 224 × 3 |
| Output classes | 2 (real, fake) |
| Modified layer | `classifier[1]`: Linear(1280 → 2) |

### Why EfficientNet-B0

Several factors:

1. **Compact**: 5.3M parameters — fits comfortably on a 12 GB GPU with batch 64.
2. **Strong ImageNet features**: EfficientNet backbones are widely used and well-validated.
3. **Good accuracy/size trade-off**: outperforms ResNet18 while using fewer parameters.
4. **Efficient inference**: fast enough for deployment considerations.

Previous experiments (V1, V2) used ResNet18. EfficientNet-B0 was chosen for V3 after comparing architectures.

## Training configuration

| Parameter | Value |
|---|---|
| Optimizer | AdamW |
| Learning rate | 1e-4 |
| Weight decay | 1e-4 |
| Scheduler | CosineAnnealingLR (T_max=30) |
| Loss | CrossEntropyLoss, label_smoothing=0.1 |
| Batch size | 64 |
| Max epochs | 30 |
| Early stopping | patience=7 |
| Mixed precision | Yes (torch.cuda.amp) |
| Seed | 42 |

### Rationale for each choice

**Optimizer: AdamW**

AdamW decouples weight decay from the gradient update. It works well with pretrained backbones and small datasets.

**Learning rate: 1e-4**

Fine-tuning with a small LR preserves pretrained features while adapting to the task. Larger LRs (1e-3) caused instability in initial experiments.

**Scheduler: CosineAnnealingLR**

Smooth LR decay over 30 epochs avoids abrupt changes. Final LR is 1e-6.

**Loss: CrossEntropyLoss with label_smoothing=0.1**

With only 76 physical cards, memorization is a real risk. Label smoothing prevents the model from becoming overconfident on training samples and improves calibration.

**Batch size: 64**

Largest that fits comfortably in 12 GB VRAM with mixed precision.

**Mixed precision**

RTX 3060 supports fp16 tensor cores. Autocast + GradScaler speed up training by ~1.5× without accuracy loss.

**Early stopping patience: 7**

Graceful stop if validation stops improving for 7 epochs.

## Model selection criterion

The best checkpoint is selected by **cross-domain F1**:

```python
score = (F1_iphone + F1_oneplus) / 2