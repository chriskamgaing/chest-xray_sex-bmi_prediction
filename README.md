# Predicting Sex and BMI from Chest X-Rays

**A Masters-level computer vision project on the BUCV26 private Kaggle competition.**

Final private leaderboard: **4th of 20**, composite score **0.5359**. This repository contains the final submission notebook, a 10-minute video walkthrough, and supporting scripts.

---

## The project in one paragraph

This project predicts patient sex (binary classification) and body mass index (regression) from chest X-ray images, evaluated through a private Kaggle competition. The final model is a ConvNeXt-Tiny with the early stages frozen, achieving an area under the receiver operating characteristic curve of 0.92 for sex classification and a body mass index root-mean-squared error of 1.94 on out-of-fold validation, corresponding to a Kaggle composite of 0.5307 on the public leaderboard and 0.5359 on the private leaderboard (rank 4 of 20). The most significant improvements came not from architectural changes but from methodological corrections — identifying patient-level data leakage in the validation split, and constraining transfer learning to prevent capacity-driven overfitting on a hundred-patient dataset. Fairness analysis of the final model revealed subgroup error patterns that motivated an explicit non-deployment recommendation.

---

## Key techniques

- PyTorch (from-scratch multilayer perceptron and convolutional neural network baselines)
- Transfer learning: ResNet-18, DenseNet-121, ConvNeXt-Tiny (torchvision, ImageNet-pretrained)
- Patient-grouped 5-fold cross-validation (`GroupKFold`) to eliminate data leakage
- Selective layer freezing for fine-tuning on small datasets
- Test-time augmentation and 5-fold ensemble inference
- Gradient-weighted Class Activation Mapping (Grad-CAM) for explainability
- Fairness analysis across sex and body-mass-index subgroups

---

## Repository contents

```
├── notebook/
│   └── bucv26_final.ipynb          Final submission notebook (Sections 1–7)
├── scripts/
│   └── video_script.md             10-min video presentation script
├── requirements.txt                Python dependencies
├── LICENSE                         MIT
└── README.md                       This file
```

---

## Notebook structure

The final notebook is organised into seven sections:

1. **Introduction** — problem framing and business case.
2. **Exploratory Data Analysis and Preprocessing** — dataset inspection, bias analysis, preprocessing pipeline.
3. **Baseline Model** — from-scratch multilayer perceptron establishing a starting benchmark.
4. **State-of-the-Art Architecture Analysis** — literature review of five relevant convolutional and transformer architectures, drawing on Bahram et al. (2026) as the primary benchmark reference.
5. **Systematic Experimentation** — twelve documented experiments tracing the model's development, including the discovery of patient-level data leakage (5.7), the corrective GroupKFold implementation (5.8), the ResNet-18 overfitting failure (5.9), the compound-augmentation experiment that surfaced the variable-confounding problem (5.10), and the final ConvNeXt-Tiny result (5.11).
6. **Final Model Evaluation and Explainability** — full end-to-end retraining of the chosen final model, classification and regression metrics, Grad-CAM visualisations, and subgroup fairness analysis.
7. **Conclusion and Reflection** — deployment recommendation and required safeguards.

---

## Reproducibility

The BUCV26 dataset is a **private Kaggle competition dataset** and is not included in this repository. Reproducing the notebook end-to-end requires access to the competition data.

To run the notebook locally or on Google Colab:

1. Obtain the BUCV26 competition data via Kaggle (requires competition access).
2. Place the data at the paths expected in Section 1 of the notebook (or update the paths accordingly).
3. Install dependencies: `pip install -r requirements.txt`
4. Run the notebook.

The chosen final model (Section 6) is fully re-runnable end-to-end. Sections 5.1–5.10 lift code from the original development notebooks with their saved outputs preserved; they are documentation of prior runs rather than a live pipeline and are not guaranteed to execute sequentially under "Restart & Run All" — this is explained in the Section 5 introduction.

A single GPU (Google Colab T4 or equivalent) is sufficient. Full Section 6 retraining takes approximately 60 minutes.

---

## Video walkthrough

A 10-minute video presentation walking through the methodology, key experiments, final results, and critical reflection is available here: **[OneDrive link — [PLACEHOLDER: paste your OneDrive share URL here]]**

*(Ensure the OneDrive share setting is "Anyone with the link can view" before publishing.)*

---

## Reflections and limitations

Portfolio projects benefit from honest limitations sections; this project has several worth stating explicitly:

- **Training data spans only 100 unique patients.** Despite the 1,000-image count, the effective demographic diversity is far narrower. This constrains external validity substantially.
- **Fairness findings preclude deployment.** The model detects females at 90% recall but males at only 68%, and the minority sex is almost entirely undetected within the overweight body-mass-index tier. Aggregate accuracy of ~82% conceals these structured errors.
- **The public-to-private leaderboard climb (rank 6 → 4)** suggests the methodological rigour applied here — patient-grouped cross-validation, selective freezing, single-variable experimentation — generalised better than heavier-fitted alternatives.

Before responsible deployment the model would require external validation on a substantially larger, multi-site, demographically balanced cohort; subgroup performance thresholds agreed with clinicians; calibration and a confidence-based abstention mechanism; and prospective testing against established measurement methods. These are detailed in Section 7 of the notebook.

---

## Author

**Christian Kamgaing** — Masters in Data Science and Artificial Intelligence at Bournemouth University.

- GitHub: [@chriskamgaing] (https://github.com/chriskamgaing)
- LinkedIn: Christian Kamgaing (https://www.linkedin.com/in/chriskamgaing/)
- Kaggle: [@christianatbu] (https://www.kaggle.com/christianatbu)

---

## Licence

Code is released under the [MIT Licence](LICENSE). The BUCV26 competition dataset is not included and is subject to Kaggle's competition terms.

---

## Acknowledgements

Primary architecture benchmark reference: Bahram, R. et al. (2026). *A comparative analysis of deep learning architectures for chest X-ray image classification*, Charmo Journal of Natural Sciences 2(1), Article 2. DOI: 10.31530/cjnst.2026.2.1.2
