# BUCV26 — 10-Minute Video Presentation Script (Academic Register · Experiment-Focused)

**Format:** Talking head + notebook screen-share split.
**Convention:**
- 🎙️ **[CAM]** = speak to camera (talking head dominant).
- 🖥️ **[SHARE → cell N]** = notebook screen-share dominant; scroll to the indicated cell/section.
- *(stage directions)* are not spoken.
- Target pace ≈ 140 words/min.

**Total budget: 10:00.** This version deliberately minimises setup and the baseline in order to deepen the experiments (Section 3) and protect the fairness reflection (Section 5). All four brief-required elements — methodology, key experiments, results, critical reflection — remain present.

---

## 0 · Opening — 0:00–0:20 (≈20s)

🎙️ **[CAM]**

Good [morning/afternoon]. My name is [your name]. This presentation documents the prediction of patient sex and body mass index from spinal X-ray images. Its central finding is methodological: the largest performance gains arose not from greater model capacity, but from correcting two methodological failures — a form of data leakage, and an overfitting failure in transfer learning. I will therefore concentrate on the experiments and their findings.

---

## 1 · The problem and the data — 0:20–0:55 (≈35s)

🖥️ **[SHARE → Section 2 EDA, patient-count output]**

The task has two heads: sex, a binary classification scored by area under the receiver operating characteristic curve; and body mass index, a regression scored by normalised root-mean-squared error. The decisive property of the data is visible here: the one thousand training images derive from only about one hundred unique patients, each contributing many images. This single fact — one hundred patients, not one thousand independent samples — constrained nearly every decision that followed, governing both how the data could be partitioned and how much model capacity was defensible.

---

## 2 · Baseline and the methodological core — 0:55–3:00 (≈125s)

🖥️ **[SHARE → Section 3 baseline, briefly]**

The from-scratch baseline, a multilayer perceptron, achieved ninety-two per cent validation accuracy but only 0.196 on Kaggle. That discrepancy is the entire problem in miniature, and the experiments exist to diagnose it.

🖥️ **[SHARE → 5.7, cell 81–83]**

The diagnosis arrives in experiment 5.7. Introducing a fixed random seed — intended only to ensure reproducibility — produced a contradiction: validation accuracy rose to ninety-nine per cent while the Kaggle score fell. A fold-level inspection revealed the cause. Identical patient identifiers were present in both the training and validation partitions. Because each patient contributes many images, a naive image-level split placed different images of the same patient on both sides of the boundary. The model was not learning anatomy; it was recognising patients it had already seen. The ninety-nine per cent figure measured memorisation, not generalisation.

🎙️ **[CAM]**

This is the most consequential moment in the project, because it explains the entire gap between local validation and leaderboard performance.

🖥️ **[SHARE → 5.8, cell 84]**

Experiment 5.8 is the correction. Patient-grouped cross-validation, via GroupKFold, guarantees that every image of a given patient remains within a single partition, so the validation set contains only patients the model has never seen. Combined with submitting raw probabilities rather than thresholded labels — necessary because the area under the curve is computed from the ranking of predictions, which thresholding destroys — this raised the composite to 0.389. For the first time, the validation metric was a trustworthy estimate of generalisation, and every later experiment is measured against it.

---

## 3 · Key experiments — 3:00–6:30 (≈210s)

🖥️ **[SHARE → 5.9, cell 93]**

With an honest baseline established, transfer learning was the natural next step: a ResNet-18 pretrained on ImageNet, fully fine-tuned. It produced the project's lowest score, 0.22. The validation area under the curve was 0.9999, yet the test predictions were severely degraded, including physiologically impossible body mass index values. The explanation is capacity: eleven million trainable parameters against one hundred patients is a ratio that permits the model to memorise the training distribution even after the leakage is corrected. Transfer learning was not the failure — *unconstrained* transfer learning was.

🖥️ **[SHARE → 5.10, cell 103 and analysis]**

Experiment 5.10 is the most instructive in the project, and I want to give it proper attention. The intention was to combine several established generalisation techniques and benefit from their compound effect. Six modifications were introduced at once: dataset-specific normalisation, horizontal flipping, rotation, colour jitter, weight decay, and test-time augmentation. The composite fell to 0.36.

*(let the 5.10 analysis cell remain on screen)*

The result itself is less important than what it exposed. Because six variables were altered simultaneously, no single one could be credited or blamed — the cross-validation differences sat within the standard-deviation envelope of the previous experiment. This is the variable-confounding problem: when several factors change together, the experiment yields no attributable evidence about any of them.

🎙️ **[CAM]**

Two lessons followed, and both shaped the remainder of the project. First, experiments must isolate a single variable if their results are to be interpretable. Second, augmentation must be justified by the domain rather than imported wholesale from natural-image pipelines. Horizontal flipping is a routine default for natural images, but a chest X-ray is not laterally symmetric — the heart lies on the left, and the lobar structure differs between sides — so flipping may erase the very asymmetry the model depends upon. The disappointing score was, in effect, the cost of learning to respect experimental discipline.

🖥️ **[SHARE → 5.11, cell 110 and result]**

The final experiments applied that discipline by changing one factor at a time. Experiment 5.11 used ConvNeXt-Tiny, but with selective freezing: the early stages were frozen, and only the final stage and the new classification head were fine-tuned. The reasoning is a direct response to the ResNet failure — by freezing the early layers, the pretrained ImageNet features are preserved rather than overwritten, so the limited data only has to adapt the high-level representations. This produced 0.531, the highest score in the project and its single largest improvement.

🖥️ **[SHARE → 5.12, cell 118 and result]**

Experiment 5.12 applied the identical procedure to DenseNet-121, which achieved 0.44. This serves as a controlled comparison: with every other factor held constant, the difference is attributable to architecture alone, confirming that the gain in 5.11 came from the architecture and not merely from the freezing strategy. The contrast with 5.10 is deliberate — these final experiments are interpretable precisely because they each varied one thing.

---

## 4 · Final model and results — 6:30–7:30 (≈60s)

🖥️ **[SHARE → 6.3 loss curves → 6.5 confusion/ROC → 6.6 scatter, scrolling as you speak]**

The final model is ConvNeXt-Tiny with selective freezing, retrained and evaluated end-to-end in Section 6. The loss curves show bounded, controlled overfitting rather than the divergence seen with ResNet. On out-of-fold predictions, sex classification reaches an area under the curve of 0.92 and accuracy of eighty-two per cent; body mass index is predicted to within roughly 1.5 units, though the scatter shows the expected regression toward the mean at the extremes.

🖥️ **[SHARE → 6.7 Grad-CAM grid]**

Gradient-weighted class activation mapping confirms that correct predictions attend to anatomically relevant regions, whereas the failures — several from a single atypical patient — attend elsewhere. The final private leaderboard score is 0.531.

---

## 5 · Critical reflection and deployment — 7:30–9:40 (≈130s)

🖥️ **[SHARE → 6.8 fairness analysis]**

The aggregate metrics, however, are not a complete evaluation, and the fairness analysis is where this project's most important caution lies. Disaggregating performance by subgroup reveals that the errors are systematic, not random.

The model correctly identifies ninety per cent of female patients but only sixty-eight per cent of male patients — a recall disparity of twenty-two percentage points, with the model defaulting toward the majority class when uncertain. Body mass index predictions are mis-calibrated in opposing directions for each sex, differing by approximately one unit. And within overweight patients, the minority sex is almost entirely undetected, even though aggregate accuracy for that group appears acceptable — a disparity that headline accuracy alone would entirely conceal.

🎙️ **[CAM]**

My recommendation is therefore explicit: this model should not be deployed. The reservation rests not on the aggregate metrics, which are respectable, but on the structure of the errors, which fall precisely along the sex and body-composition lines the model is meant to estimate. A model whose mistakes track the protected attribute it predicts is not safe to deploy regardless of its average accuracy.

Before responsible use, four conditions would be required: external validation on a substantially larger, multi-site, and demographically balanced cohort; subgroup performance thresholds agreed with clinicians; calibration analysis together with a confidence-based mechanism for abstaining on uncertain cases; and prospective testing against established measurement methods. A training base of one hundred patients cannot support clinical claims.

---

## 6 · Conclusion — 9:40–10:00 (≈20s)

🎙️ **[CAM]**

In conclusion, the decisive improvements in this project came from methodological rigour rather than model capacity: correcting patient-level leakage, restraining excessive capacity, and isolating single variables during experimentation. The final model is genuinely promising, but the fairness analysis is precisely why it must remain a proof of concept rather than a deployable system. Thank you.

---

## Appendix — timing cheat sheet

| Section | Target end | Running topic | Δ vs balanced script |
|---|---|---|---|
| 0 Opening | 0:20 | Thesis: methodology over capacity | −15s |
| 1 Problem & data | 0:55 | 100 patients (setup minimised) | −40s |
| 2 Baseline & core | 3:00 | One-line baseline → leakage → GroupKFold (0.389) | −20s |
| 3 Key experiments | 6:30 | ResNet fail → **5.10 deep dive** → **ConvNeXt win** → DenseNet control | **+75s** |
| 4 Final results | 7:30 | Visuals carry it; minimal narration | −30s |
| 5 Reflection | 9:40 | Fairness → non-deployment + four conditions | protected |
| 6 Conclusion | 10:00 | Recapitulation | — |

**Word count:** ≈1,480 spoken words ≈ 10:05 at 140 wpm. Marginally over; if consistently long, shorten the Section 4 narration further (the visuals are self-explanatory) or tighten the ResNet paragraph in Section 3.

**Pacing guidance for the deep-dive sections:** In Section 3, slow down on 5.10 and 5.11 — these are the assessed core. The screen-share gives you natural pauses; let the 5.10 analysis cell and the ConvNeXt result sit on screen for a beat before speaking. Resist the urge to fill silence; a marker reading the cell while you pause is a feature, not dead air.

**Protected content:** Sections 2 (leakage/GroupKFold), 3 (experiments), and 5 (fairness) carry the marks for methodology, experimental rigour, and critical evaluation. If anything must be cut live, take it from Sections 1 or 4, never these three.
