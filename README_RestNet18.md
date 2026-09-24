# 🃏 Pokémon Card Authenticity Detection

## Computer Vision for Counterfeit Pokémon Card Detection using ResNet18

A computer vision project that investigates whether a deep learning model can distinguish **authentic Pokémon cards from counterfeit cards** using photographs of physical cards.

The project focuses not only on achieving high classification accuracy, but also on evaluating **generalization, data leakage, model interpretability, and real-world limitations**.

---

## 📌 Project Overview

Counterfeit trading cards can contain subtle visual differences from authentic cards. These differences may appear in:

- Printing quality
- Color and tonal characteristics
- Borders and typography
- Card texture
- Holographic patterns
- Illustration details
- Text and symbols
- Small visual inconsistencies

This project explores whether a convolutional neural network can learn these differences from photographs.

The main question is:

> **Can a deep learning model learn to distinguish authentic Pokémon cards from counterfeit cards, and does that knowledge generalize to cards that were not present during training?**

Rather than treating the problem as a simple image classification exercise, the project evaluates the complete machine learning pipeline, from dataset construction to independent external validation and model interpretability.

---

# 🎯 Objectives

### Primary objective

Build a binary image classification model capable of distinguishing:

- `REAL` — authentic Pokémon card
- `FAKE` — counterfeit Pokémon card

### Secondary objectives

The project also aims to:

1. Build a dataset using physical Pokémon cards.
2. Prevent data leakage between training, validation and test sets.
3. Train a CNN using transfer learning.
4. Evaluate performance on a held-out internal test set.
5. Evaluate generalization using an independently collected external dataset.
6. Investigate model behavior using Grad-CAM.
7. Identify possible dataset biases and shortcut learning.
8. Analyze model failures rather than focusing exclusively on accuracy.
9. Document the limitations of using computer vision for physical card authentication.

---

# 🧠 Methodology

The project follows this pipeline:

```text
Physical Pokémon Cards
        │
        ▼
Photograph Acquisition
        │
        ▼
Dataset Organization
        │
        ▼
Data Quality & Leakage Analysis
        │
        ▼
Card-Level Train / Validation / Test Split
        │
        ▼
Image Preprocessing & Augmentation
        │
        ▼
ResNet18 + Transfer Learning
        │
        ▼
Model Training
        │
        ▼
Internal Test Evaluation
        │
        ▼
Independent External Dataset
        │
        ▼
Error Analysis
        │
        ▼
Grad-CAM Interpretability
```

---

# 📊 Dataset

The initial dataset was constructed from **physical Pokémon cards**.

Each physical card can have multiple photographs. Therefore, the unit used for dataset splitting is the **physical card**, rather than the individual image.

## Initial Dataset

| Class | Physical Cards | Images |
|---|---:|---:|
| REAL | 55 | 542 |
| FAKE | 55 | 542 |
| **Total** | **110** | **1084** |

The dataset contains an equal number of authentic and counterfeit cards.

---

# 🔐 Preventing Data Leakage

One of the most important decisions in the project was avoiding image-level random splitting.

Because multiple photographs can belong to the same physical card, randomly distributing images could result in the same card appearing in both training and test sets.

For example:

```text
C032_REAL_01.jpg → TRAIN
C032_REAL_02.jpg → TRAIN
C032_REAL_03.jpg → TEST
```

In this situation, the model could potentially learn the identity or unique visual characteristics of card `C032` instead of learning characteristics related to authenticity.

To avoid this problem, the dataset was split using the **physical card identity / card pair**.

### Dataset split

| Split | Card Pairs | Images |
|---|---:|---:|
| TRAIN | 38 | 746 |
| VALIDATION | 8 | 158 |
| TEST | 9 | 180 |
| **Total** | **55** | **1084** |

The split was performed at the card level to ensure that photographs of the same physical card did not cross dataset boundaries.

> **This is an important distinction because the goal is to evaluate generalization to unseen physical cards, not recognition of previously observed cards.**

---

# 🤖 Model

The project uses **ResNet18**, a convolutional neural network architecture, through transfer learning.

The original classification layer was replaced with a binary classifier:

```text
ResNet18
   │
   ├── Convolutional layers
   ├── Residual blocks
   ├── Feature extraction
   │
   ▼
Fully Connected Layer
   │
   ├── REAL
   └── FAKE
```

### Model configuration

| Parameter | Value |
|---|---|
| Architecture | ResNet18 |
| Framework | PyTorch |
| Number of classes | 2 |
| Total parameters | 11,177,538 |
| Optimizer | AdamW |
| Learning rate | 0.0001 |
| Weight decay | 0.0001 |
| Loss function | CrossEntropyLoss |
| Scheduler | ReduceLROnPlateau |
| Maximum epochs | 15 |

The final fully connected layer was adapted for the two-class classification problem.

---

# 🏋️ Training

During training, the model showed rapid improvement.

The first epoch achieved:

```text
Training Accuracy: 67.02%
Validation Accuracy: 91.77%
```

By epoch 3:

```text
Validation Accuracy: 98.10%
```

The best checkpoint was obtained at:

```text
Epoch: 13
Validation Loss: 0.0074
Validation Accuracy: 100.00%
```

The best model was saved as:

```text
pokemon_fake_detector_v1_best.pth
```

---

# 📈 Internal Test Results

The final model was evaluated on the held-out internal test set.

### Results

| Metric | Score |
|---|---:|
| Accuracy | **99.44%** |
| Precision | **100.00%** |
| Recall | **98.89%** |
| F1 Score | **99.44%** |

These results indicate excellent performance on the internal test set.

However, this result should **not** be interpreted as proof that the model can reliably authenticate arbitrary Pokémon cards in the real world.

The internal test set consists of cards originating from the same overall data collection process.

Therefore, an additional independent evaluation was performed.

---

# 🌎 External Dataset

To investigate whether the model generalizes beyond the original dataset, an independently collected external dataset was created **after the initial model had been trained**.

This dataset contains cards that were not part of the training process.

### External dataset

| Characteristic | Value |
|---|---:|
| Physical cards | 40 |
| REAL cards | 20 |
| FAKE cards | 20 |
| Photographs | 80 |
| Photographs per card | 2 |
| English cards | 36 |
| Spanish cards | 4 |
| iPhone photographs | 40 |
| OnePlus photographs | 40 |

The external dataset was specifically designed to introduce additional variation.

It includes:

- Previously unseen physical cards
- Different acquisition conditions
- Multiple cameras
- Different card languages
- Additional visual characteristics

---

# 🔬 External Evaluation

The external dataset is treated as a separate evaluation domain rather than being incorporated into training.

This allows the following question to be investigated:

> **Does the model generalize from the original dataset to previously unseen cards?**

These errors are particularly valuable because they provide information that a single accuracy score cannot.

The external evaluation is therefore used for **error analysis and robustness assessment**, rather than simply reporting another performance number.

> ⚠️ The external evaluation should be interpreted separately from the internal test results. High performance on the original test set does not guarantee equivalent performance on unseen cards.

---

# 🧪 External Evaluation

The external dataset provides a more demanding evaluation of model generalization.

Overall performance
Metric	Result
Images evaluated	80
Correct predictions	72
Incorrect predictions	8
Accuracy	90.00%
Macro Precision	90.92%
Macro Recall	90.00%
Macro F1	89.94%

The difference between the internal and external evaluations is significant:

Internal Test
     │
     ▼
99.44% Accuracy
     │
     │
     ▼
External Dataset
     │
     ▼
90.00% Accuracy

This drop in performance suggests that the model does not generalize perfectly to previously unseen cards and acquisition conditions.

This is an important finding of the project.

---

# 🔍 External Confusion Matrix

The external dataset contained 40 authentic and 40 counterfeit images.

Actual \ Predicted	REAL	FAKE
REAL	39	1
FAKE	7	33

Therefore:

39/40 authentic images were correctly classified.
33/40 counterfeit images were correctly classified.
7 counterfeit images were incorrectly classified as authentic.
1 authentic image was incorrectly classified as counterfeit.

The main weakness of the model in the external dataset is therefore false negatives: counterfeit cards classified as REAL.

---

# 🎯 Performance by Class
Class	Correct	Total	Recall
REAL	39	40	97.50%
FAKE	33	40	82.50%

The model performs substantially better at identifying authentic cards than counterfeit cards in the external dataset.

This asymmetry is important because, for a counterfeit detection system, false negatives are particularly relevant.

A false negative occurs when:

Actual card: FAKE
        ↓
Model prediction: REAL

This type of error would be more concerning in a real authentication scenario than a false positive.

---

# 📷 Performance by Camera

Each external card was photographed using both an iPhone and a OnePlus device.

Interestingly, both cameras produced exactly the same overall image-level accuracy:

Camera	Correct	Total	Accuracy
iPhone	36	40	90.00%
OnePlus	36	40	90.00%

This result does not provide evidence of a camera-specific performance difference in this particular experiment.

However, the sample size is still too small to conclude that the model is completely invariant to camera characteristics.

---

# 🃏 Performance by Physical Card

Because each physical card was photographed twice, it is also possible to evaluate performance at the card level.

Out of the 40 external physical cards:

35 cards
│
├── Both photographs correctly classified
│
└── 87.5% of physical cards

Five physical cards had at least one incorrect prediction.

These included:

T005
T006
T009
T010
T035

This provides another perspective that is not visible from image-level accuracy alone.

---

# ❌ Error Analysis

The eight incorrect image-level predictions were:

Card	Camera	Actual	Prediction	Confidence
T005	iPhone	FAKE	REAL	88.56%
T005	OnePlus	FAKE	REAL	89.98%
T006	iPhone	FAKE	REAL	65.70%
T006	OnePlus	FAKE	REAL	83.19%
T009	OnePlus	FAKE	REAL	53.05%
T010	iPhone	FAKE	REAL	99.38%
T010	OnePlus	FAKE	REAL	98.98%
T035	iPhone	REAL	FAKE	53.44%

A particularly interesting case is T010.

The model classified the counterfeit card as authentic with approximately:

iPhone  → REAL 99.38%
OnePlus → REAL 98.98%

This indicates that the error is not simply caused by uncertainty in the image.

The model was highly confident in an incorrect prediction.

This is an important example of why:

Model confidence should not be interpreted as authentication certainty.

---

# 👁️ Model Interpretability with Grad-CAM

To investigate what the model is looking at, Grad-CAM was used.

Grad-CAM produces a visual representation of the regions that contribute most strongly to a prediction.

Conceptually:

Input Image
     │
     ▼
  ResNet18
     │
     ▼
Prediction
     │
     ▼
  Grad-CAM
     │
     ▼
Important Image Regions

This makes it possible to investigate whether the model is focusing on visually meaningful characteristics.

---

# 🔬 Example: T005

T005 is a counterfeit card that was incorrectly classified as authentic by both cameras.

iPhone
Actual: FAKE
Prediction: REAL
Confidence: 88.56%
OnePlus
Actual: FAKE
Prediction: REAL
Confidence: 89.98%

The Grad-CAM visualizations show that the model's attention extends strongly toward the lower region of the card.

This is interesting because the model does not appear to be explicitly identifying a single obvious counterfeit characteristic.

Instead, the prediction may be influenced by broader visual patterns.

T005 observation
              Original Card
                    │
                    ▼
              ResNet18
                    │
                    ▼
             Prediction
                    │
             ┌──────┴──────┐
             │             │
          iPhone         OnePlus
             │             │
          REAL 88.6%    REAL 90.0%
             │             │
             └──────┬──────┘
                    ▼
                 Grad-CAM
                    │
                    ▼
        Strong activation in lower
              card region

This example illustrates why model interpretability is useful: the model can be confidently wrong, and the attention map provides additional information for investigating the reason.

---

# ⚠️ Potential Biases and Shortcut Learning

A major concern in this project is shortcut learning.

A neural network does not inherently understand what makes a Pokémon card authentic.

It learns statistical patterns that help minimize the training loss.

Therefore, the model could potentially learn characteristics that correlate with authenticity but are not actually reliable indicators of counterfeiting.

Potential sources of bias include:

Watermarks

The Pokémon watermark or other recurring visual elements could become shortcuts for classification.

Card identity

Specific cards may contain unique artwork, typography, colors or layouts.

The card-level split reduces this risk but does not completely eliminate the possibility of card-specific visual correlations.

Camera characteristics

Different cameras can introduce:

Color differences
Sharpness differences
Exposure differences
Noise patterns
White balance differences

The external experiment produced equal overall accuracy between the two cameras, but this sample is not large enough to eliminate camera-related bias.

Lighting

Holographic and reflective surfaces can behave very differently depending on illumination and camera angle.

Language

The external dataset includes both English and Spanish cards.

Language-related visual differences may influence predictions even though language itself is not a criterion for authenticity.

---

# ✨ Holographic Cards

Holographic cards represent a particularly interesting challenge.

Authentic and counterfeit holographic cards can behave differently under light, but photographic conditions can strongly influence their appearance.

The project includes holographic examples with different visual characteristics, including:

Silver holographic effects
Purple holographic effects
Yellow holographic effects
Other reflective patterns

However, the number and diversity of counterfeit holographic examples remains limited.

Therefore:

Performance on holographic cards should not be generalized to the entire population of counterfeit holographic Pokémon cards.

Additional holographic samples are planned for future dataset expansion.

---

# 📊 Grad-CAM Concentration Analysis

In addition to visual inspection of Grad-CAM maps, a concentration metric based on the activation distribution was calculated.

The resulting analysis suggests that incorrect predictions do not necessarily correspond to a simple "low attention" pattern.

For example, some incorrect predictions show concentrated activation in specific regions, while others distribute attention more broadly.

This supports an important observation:

The model can produce a highly confident prediction even when its visual attention does not necessarily correspond to an obvious human-interpretable authentication feature.

Grad-CAM is therefore treated as an interpretability tool rather than as definitive evidence of what the model "understands."

---

# 🔎 Error Analysis

One of the main goals of the project is understanding **why the model fails**.

Incorrect predictions from the external dataset are analyzed individually.

For example, some cards produced predictions that were difficult to explain from the perspective of human visual inspection.

The analysis investigates whether errors could be related to:

- Card artwork
- Background characteristics
- Lighting
- Camera differences
- Language
- Holographic effects
- Printing characteristics
- Card layout
- Image composition
- Specific card identities

The objective is to distinguish between:

```text
Useful visual features
        vs.
Dataset-specific shortcuts
```

---

# 🧪 Why External Validation Matters

The project demonstrates an important principle in machine learning:

> **A high test score is not necessarily evidence of real-world generalization.**

The internal test set produced:

```text
99.44% Accuracy
```

which is excellent.

However, the external dataset introduced a new challenge:

```text
Unseen physical cards
        +
Different photographs
        +
Different cameras
        +
Different languages
        +
Different card characteristics
```

This makes the external evaluation a more realistic test of robustness.

The project therefore treats model evaluation as a progression:

```text
Training Performance
        ↓
Internal Validation
        ↓
Held-out Test Set
        ↓
Independent External Dataset
        ↓
Error Analysis
        ↓
Interpretability
```

---

# 🛠️ Technology Stack

The project was developed using:

- Python 3.10.14
- PyTorch 2.5.1
- Torchvision 0.20.1
- Pillow 9.4.0
- Pandas 2.3.3
- Scikit-learn 1.7.2
- Jupyter Notebook
- CUDA
- NVIDIA RTX 3060 12GB

---

# 📁 Project Structure

The repository is organized around reproducibility and separation of responsibilities. The initial public version focuses on the documented experiment and the clean training notebook; additional scripts and experiments can be added as the project evolves.

```text
pokemon-card-authenticity/
│
├── README.md
├── .gitignore
│
├── data/
│   └── README.md
│
├── notebooks/
│   └── 02_training.ipynb
│
├── models/
│   └── README.md
│
├── results/
│   ├── metrics/
│   ├── confusion_matrix/
│   └── gradcam/
│
└── docs/
    └── methodology.md
```

The actual repository structure may evolve as the project is documented and additional experiments are added.

---

# 🔄 Reproducibility

The objective is to make the experiment reproducible.

Important components include:

- Fixed dataset organization
- Card-level dataset splitting
- Explicit training configuration
- Saved model checkpoints
- Evaluation methodology
- External test dataset
- Model interpretability analysis
- Environment and dependency documentation

The dataset itself is not currently included in the repository because it consists of photographs of physical cards collected for this research project.

---

# 🚧 Limitations

This project is an experimental computer vision system and **is not a certified authentication system**.

Important limitations include:

1. The dataset is relatively small compared with large-scale computer vision datasets.
2. The diversity of counterfeit cards is limited.
3. Holographic counterfeit examples are still limited.
4. Photographic conditions can influence predictions.
5. Camera-specific characteristics may introduce bias.
6. Some cards may contain visual characteristics that correlate with authenticity.
7. A model confidence score does not represent a probability of real-world authentication.
8. External evaluation is still limited in size.
9. The model has not been evaluated against every Pokémon card generation, set, language or printing variation.
10. Physical characteristics that cannot be captured reliably in a photograph are outside the scope of the current system.

---

# 🚀 Future Work

Several improvements are planned.

### Dataset expansion

- Add more authentic cards.
- Add more counterfeit cards.
- Increase the number of unique card pairs.
- Add more languages.
- Increase the number of holographic cards.
- Include additional counterfeit manufacturing characteristics.

### Model improvements

Potential experiments include:

- ResNet50
- EfficientNet
- ConvNeXt
- Vision Transformers
- Different augmentation strategies
- Hyperparameter optimization
- Class-weighted training
- Ensemble models

### Robustness

Future experiments could evaluate:

- Different lighting conditions
- Different backgrounds
- Different cameras
- Different distances
- Rotation
- Perspective changes
- Cropping
- Low-resolution photographs

### Interpretability

Further Grad-CAM analysis could investigate:

- Which regions consistently influence predictions.
- Whether the model relies on artwork.
- Whether it uses borders or text.
- Whether it detects printing characteristics.
- Whether holographic regions influence predictions.
- Whether the model exhibits camera-specific shortcuts.

---

# 📌 Key Lessons

This project reinforced several important machine learning principles.

### 1. Dataset design matters as much as model architecture

A sophisticated model cannot compensate for a poorly designed evaluation strategy.

### 2. Data leakage can produce misleadingly high performance

When multiple images belong to the same physical object, splitting by image can make a model appear much better than it actually is.

### 3. Accuracy is not enough

A model can achieve excellent test performance and still fail on unseen data.

### 4. External validation is essential

An independent dataset provides a more realistic estimate of generalization.

### 5. Interpretability matters

Grad-CAM can reveal whether a model appears to be using meaningful visual features or potentially relying on shortcuts.

### 6. Failure analysis is valuable

Incorrect predictions often teach more about a model than correctly classified examples.

---

# 🧭 Project Status

**Current status: Experimental / Active Development**

### Completed

- [x] Physical card dataset collection
- [x] Real/fake dataset organization
- [x] Card-level train/validation/test split
- [x] Data leakage prevention
- [x] ResNet18 training
- [x] Internal test evaluation
- [x] External dataset collection
- [x] External evaluation
- [x] Initial error analysis
- [x] Grad-CAM analysis
- [x] Bias and limitation analysis

### In progress

- [ ] Expand external dataset
- [ ] Add more holographic examples
- [ ] Improve external validation
- [ ] Compare additional architectures
- [ ] Document complete experimental history
- [ ] Add visual results to repository
- [ ] Publish reproducible training pipeline

---

# 👤 Author

**Jonathan Paredes**

Computer Science / Informatics Engineer  
Chile

Areas of interest:

- Data Engineering
- Machine Learning
- Computer Vision
- Data Analytics
- MLOps

---

# 📜 Disclaimer

This project is an educational and experimental machine learning project.

The model's predictions should **not be considered definitive proof of card authenticity**.

Professional authentication may require physical inspection of characteristics that cannot be reliably captured through photographs alone.

---

# ⭐ Final Note

The purpose of this project is not simply to build a model that achieves a high accuracy score.

The broader objective is to investigate the complete machine learning problem:

```text
Can we collect meaningful data?
        ↓
Can we prevent leakage?
        ↓
Can we train a useful model?
        ↓
Does it generalize?
        ↓
Where does it fail?
        ↓
What is the model actually looking at?
        ↓
What are its limitations?
```

The goal is therefore to move from:

> **"The model gets 99% accuracy."**

to:

> **"We understand how the model was trained, how it was evaluated, where it generalizes, where it fails, and what factors may influence its decisions."**

That distinction is at the core of this project.