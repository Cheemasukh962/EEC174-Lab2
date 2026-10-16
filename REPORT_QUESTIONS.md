# Lab 2: Written report questions

Answer the following questions in your written report. For questions about model performance, refer to results from your own notebook and report the relevant metrics.

## 1. Data and labels

1. **Prediction task:** What does one row of the dataset represent? What are the three classes predicted in this lab, and how are they constructed from the BSA and DTA annotations?

   **Answer:** Each row is one breath from one patient, described by numbers measured from its flow and pressure. The three classes are 0 = normal, 1 = breath stacking (BSA) and 2 = double trigger (DTA). They are built as `bsa * 1 + dta * 2`.

2. **Ambiguous labels:** The notebook removes breaths annotated as both BSA and DTA. Why are these rows a problem for the three-class target? Describe one alternative to removing them.

   **Answer:** A breath marked as both gives `1 + 2 = 3`, which is not one of our three classes. Instead of removing it, we could assign it to just one class, such as DTA.

3. **Patient identifiers:** Why are `patient` and `breathid` excluded from the model inputs? Does excluding them guarantee that the model will generalize to patients it has never seen? Explain.

   **Answer:** They are just ID numbers and say nothing about the breath itself, so the model could only use them to memorize patients. Removing them does not guarantee it works on new patients, because each patient's breaths still look alike. The only way to check is to test on patients the model has never seen.

## 2. Features and preprocessing

4. **Feature extraction:** Why does the lab extract features from flow and pressure waveforms before training a classifier? Identify two features that capture different aspects of a breath, and explain what each contributes.

   **Answer:** A classifier needs the same small set of numbers for every breath, but raw waveforms are long and vary in length. Two useful features:
   - `tve_tvi_ratio` (air out ÷ air in): a value well below 1 means air is trapped, a sign of breath stacking.
   - `e_time` (time spent breathing out): a very short value means the next breath started too early, a sign of double trigger.

5. **Feature scaling:** Which scaling method did you use? Why should it be fitted on the training data and then applied to the validation data, rather than fitted on both datasets together?

   **Answer:** I used `StandardScaler`. It is fitted on the training data only, so the validation data stays truly unseen. Fitting on both would let the model "peek" at the validation data and make the results look better than they really are.

## 3. Model performance

6. **Random forest:** In your own words, how does a random forest combine decision trees to make a prediction? Give one reason it is a reasonable choice for this dataset.

   **Answer:** It trains many decision trees on random parts of the data, and each tree votes for a class. The class with the most votes wins. It suits this data because these breath problems are defined by simple cut-offs (like ratio < 0.9), which is exactly what trees learn.

7. **Class-specific errors:** Examine your random forest's classification report. Which class has the lowest recall, and what kinds of mistakes does that indicate? Why might accuracy alone hide this problem?

   **Answer:** Double trigger (DTA) has the lowest recall, about 0.05, so the model misses almost every double trigger and calls them normal or BSA instead. Accuracy still looks high (about 0.89) because DTA is only about 8% of the breaths.

8. **Precision and recall:** For detecting BSA or DTA, what is the difference between a false positive and a false negative? Explain a situation in which you would prioritize recall over precision for one of these classes.

   **Answer:** A false positive is a false alarm: a normal breath flagged as BSA or DTA. A false negative is a missed problem: a real BSA or DTA breath called normal. In an ICU, catching every real DTA (high recall) matters more, because a missed problem can hurt the patient, while a false alarm only costs a quick check.

9. **Model comparison:** Compare the random forest with the other supervised-learning model you implemented. Which has the higher F1-score? Does the same model perform best for every class? Support your answer with your results.

   **Answer:** The random forest did better than logistic regression, with a macro F1 of about 0.64 versus 0.53. It was best on every class:

   | Class | Random forest F1 | Logistic regression F1 |
   |---|---|---|
   | Normal | 0.93 | 0.86 |
   | BSA | 0.90 | 0.67 |
   | DTA | 0.09 | 0.07 |

   Both models struggle with DTA.

## 4. Model selection and validation

10. **Feature selection:** Compare performance using all available breath features with performance using selected features or the lab's expert-defined rules. Did the reduced feature set improve validation performance? Why might a feature that appears useful on the training set fail to improve validation performance?

    **Answer:** The two features chosen by the χ² test (`tve`, `ep_auc`) made the results worse than using all features. The doctors' expert rules did much better, with 97% accuracy and a macro F1 of 0.94, though this was measured on the training set. A feature can look useful in training only because it matches those particular patients, and that pattern may not hold for new patients.

11. **Cross-validation:** What does stratification preserve in 5-fold cross-validation? Report your F1-score for each fold and its average. What would substantial differences between folds suggest?

    **Answer:** Stratification keeps the same mix of classes in every fold (about 72% normal, 24% BSA and 4% DTA).

    | Fold | 1 | 2 | 3 | 4 | 5 | Average |
    |---|---|---|---|---|---|---|
    | Macro F1 | 0.909 | 0.920 | 0.914 | 0.926 | 0.928 | **0.919** |

    Big differences between folds would mean the model is unstable and its score depends on which breaths it happens to train on.

12. **Patient-level validation:** If breaths from the same patient appear in both training and validation sets, how might that affect the reported performance? Describe a validation strategy for estimating performance on patients the model has not seen before.

    **Answer:** The model can partly "recognize" the patient, which makes its scores look too good. That is likely why cross-validation gave an F1 of about 0.92 but the new-patient validation set gave only about 0.64. The fix is to split by patient (for example with `GroupKFold`), so all of a patient's breaths go into either training or testing, never both.
