# Checkpoint 7 — Zero Imputation vs Median Imputation (LOOCV)
**Date:** 2026-05-18
**Project:** Severe vs Non-severe COVID-19 Classifier
**Dataset:** Proteomic and Metabolomic Characterization of COVID-19 Patient Sera (MSV000085507)

---

## 1. Goal
Test whether replacing missing peptide intensities with **zero** instead of the training-fold **median** changes classification accuracy. This was implemented in `modeling4.ipynb`, which otherwise uses the same LOOCV structure as `modeling3.ipynb`.

---

## 2. Summary

In mass spectrometry, a zero (missing) intensity means the peptide was below the instrument's detection limit — it was not detected, not that its true abundance is zero. `modeling3.ipynb` uses median imputation, which fills missing values with the median of detected intensities in the training fold. This blurs the "not detected" signal into a typical detected value.

`modeling4.ipynb` instead fills all NaN with zero (`SimpleImputer(strategy="constant", fill_value=0)`). This preserves the "not detected" distinction: a peptide with zero intensity is treated differently from one with a typical measured intensity. Theoretically, this could help the model if missingness itself is informative — for example, if a peptide is systematically detected in Severe patients but not in Non-severe patients.

Feature selection and LOOCV are identical to `modeling3.ipynb`: t-tests and Mann-Whitney U tests are run inside the loop on the 42 training patients only, top 200 peptides are selected, and the same 200 are applied to the held-out patient. The imputation from zero (not median) happens inside the sklearn pipeline, after the fold split, so there is no data leakage.

The result: accuracy and F1 are identical to median imputation. The only difference is a higher ROC-AUC (0.967 vs 0.944), suggesting the model's probability estimates are better calibrated with zero imputation — it is more confident in its correct predictions.

---

## 3. Key Results

All three models compared across LOOCV (43 folds), zero imputation:

| Model | Accuracy | Precision (Severe) | Recall (Severe) | F1 (Severe) | ROC-AUC |
|-------|----------|--------------------|-----------------|-------------|---------|
| **Logistic Regression L2** | **0.907** | **0.889** | **0.889** | **0.889** | **0.967** |
| Logistic Regression L1 | 0.814 | 0.727 | 0.889 | 0.800 | 0.927 |
| Random Forest | 0.791 | 0.846 | 0.611 | 0.710 | 0.924 |

Best model: **Logistic Regression L2** (same as median imputation).

Misclassifications (best model, 4 total):
- **False Positives** (Non-severe called Severe): Patient-group-PT, XG3
- **False Negatives** (Severe called Non-severe): XG44, XG45

These are the same two Non-severe false positives as median imputation. The two false negatives (XG44, XG45) are also identical — these are the most ambiguous Severe patients in the dataset.

Zero vs median imputation comparison (both using LOOCV, best model = LR L2):

| Pipeline | Accuracy | Precision (Severe) | Recall (Severe) | F1 (Severe) | ROC-AUC |
|----------|----------|--------------------|-----------------|-------------|---------|
| Median imputation (`modeling3.ipynb`) | 0.907 | 0.889 | 0.889 | 0.889 | 0.944 |
| **Zero imputation (`modeling4.ipynb`)** | **0.907** | **0.889** | **0.889** | **0.889** | **0.967** |

---

## 4. Important Code

### Zero imputation in model pipeline
```python
# All three models use fill_value=0 instead of strategy="median"
Pipeline([
    ("imputer", SimpleImputer(strategy="constant", fill_value=0)),
    ("scaler",  StandardScaler()),
    ("model",   LogisticRegression(penalty="l2", C=1.0, class_weight="balanced", ...)),
])
```

### Feature selection still runs inside the LOOCV loop (no leakage)
```python
for fold, (train_idx, test_idx) in enumerate(loo.split(X)):
    X_train_fold = X.iloc[train_idx]   # 42 patients — used for feature selection
    X_test_fold  = X.iloc[test_idx]    # 1 patient  — never influences feature selection

    # Vectorised t-test and Mann-Whitney U on all 101,461 peptides at once
    _, t_pvals  = ttest_ind(X_sv_arr, X_nsv_arr, axis=0, equal_var=False, nan_policy="omit")
    _, mw_pvals = mannwhitneyu(X_sv_arr, X_nsv_arr, axis=0, alternative="two-sided", nan_policy="omit")

    selected = stats_fold.dropna(subset=["mannwhitney_pvalue"]).head(200)["Peptide"].tolist()
    # Imputation with 0 happens inside pipe.fit() — after the split, no leakage
    pipe.fit(X_train_fold[selected], y_train_fold)
```

---

## 5. To Do Next / Blockers

### To Do Next
- [ ] Implement a Neural Network model (listed in `project_plan.md` as a proposed model, not yet done)
- [ ] Investigate why XG44 and XG45 are consistently misclassified as Non-severe across all pipelines

### Blockers
- None.

---

## 6. File Index

| Filename | Description |
|----------|-------------|
| `modeling4.ipynb` | LOOCV pipeline with zero imputation and vectorised feature selection |
| `models/modeling4_loocv_comparison.tsv` | LOOCV metrics for all 3 models (zero imputation) |
| `models/modeling4_loocv_predictions.tsv` | Per-patient predictions from LR L2 |
| `models/modeling4_final_model.joblib` | LR L2 retrained on all 43 patients with zero imputation |
| `models/modeling4_best_model_name.txt` | Best model name (logistic_regression_l2) |
| `models/modeling4_selected_peptides.txt` | 200 peptides selected from full 43-patient dataset |
| `checkpoints/checkpoint7.md` | This file |
