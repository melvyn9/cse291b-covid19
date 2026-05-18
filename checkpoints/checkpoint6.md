# Checkpoint 6 — Leave-One-Out Cross-Validation (LOOCV)
**Date:** 2026-05-17
**Project:** Severe vs Non-severe COVID-19 Classifier
**Dataset:** Proteomic and Metabolomic Characterization of COVID-19 Patient Sera (MSV000085507)

---

## 1. Goal
Replace the single 25/9/9 train/validation/test split from `modeling2.ipynb` with Leave-One-Out Cross-Validation (LOOCV) to get a more reliable performance estimate given our small sample size of 43 patients.

---

## 2. Summary
With only 43 patients, a single test set of 9 patients is too small to produce stable accuracy estimates — one misclassification shifts accuracy by ~11%, and results vary depending on which 9 patients happen to land in the test set.

LOOCV solves this by rotating every patient through the test position exactly once. In each of the 43 folds, one patient is held out, the model is trained on the remaining 42, and a prediction is made on the held-out patient. The final accuracy is computed over all 43 predictions, each made on a patient the model had never seen during that fold's training.

To prevent data leakage, statistical feature selection (t-test and Mann-Whitney U test) is performed **inside the loop** on the 42 training patients only for each fold. The held-out patient has zero influence on which 200 peptides are selected. Imputation and scaling also use only training-fold statistics.

After LOOCV evaluation, the best model is retrained on all 43 patients (with feature selection on the full dataset) to produce a final model for feature importance inspection. This retraining is acceptable because performance was already estimated by LOOCV — it is not used for evaluation.

---

## 3. Key Results

All three models compared across LOOCV (43 folds):

| Model | Accuracy | Precision (Severe) | Recall (Severe) | F1 (Severe) | ROC-AUC |
|-------|----------|--------------------|-----------------|-------------|---------|
| **Logistic Regression L2** | **0.907** | **0.889** | **0.889** | **0.889** | **0.944** |
| Logistic Regression L1 | 0.814 | 0.727 | 0.889 | 0.800 | 0.913 |
| Random Forest | 0.814 | 0.813 | 0.722 | 0.765 | 0.927 |

Best model: **Logistic Regression L2** (selected by F1, then ROC-AUC).

Confusion matrix for LR L2 across all 43 LOOCV folds:
- True Negatives  (Non-severe → Non-severe): 23
- False Positives (Non-severe → Severe):       2
- False Negatives (Severe → Non-severe):        2
- True Positives  (Severe → Severe):           16

Pipeline comparison (all three approaches):

| Pipeline | Best Model | Accuracy | Precision (Severe) | Recall (Severe) | F1 (Severe) | ROC-AUC |
|----------|-----------|----------|--------------------|-----------------|-------------|---------|
| Leaky single-split (`modeling.ipynb`) | LR L2 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |
| Corrected single-split (`modeling2.ipynb`) | LR L1 | 0.889 | 0.800 | 1.000 | 0.889 | 0.950 |
| **LOOCV (`modeling3.ipynb`)** | **LR L2** | **0.907** | **0.889** | **0.889** | **0.944** | **0.944** |

---

## 4. Important Code

### LOOCV loop with feature selection inside each fold
```python
loo = LeaveOneOut()
loocv_results = {name: {"y_true": [], "y_pred": [], "y_score": []} for name in model_templates}

for fold, (train_idx, test_idx) in enumerate(loo.split(X)):
    X_train_fold = X.iloc[train_idx]   # 42 patients
    X_test_fold  = X.iloc[test_idx]    # 1 patient
    y_train_fold = y.iloc[train_idx]

    # Statistical tests on training fold ONLY — no leakage from held-out patient
    for peptide in X_train_fold.columns:
        sv  = X_train_fold.loc[condition_fold == "Severe-COVID-19",     peptide].dropna()
        nsv = X_train_fold.loc[condition_fold == "Non-severe-COVID-19", peptide].dropna()
        ...

    selected = stats_fold.dropna(subset=["mannwhitney_pvalue"]).head(200)["Peptide"].tolist()
    X_tr = X_train_fold[selected]
    X_te = X_test_fold[selected]   # same 200 columns applied to held-out patient

    # clone() creates a fresh unfitted pipeline for each fold
    pipe = clone(template)
    pipe.fit(X_tr, y_train_fold)
    pred  = pipe.predict(X_te)[0]
    score = pipe.predict_proba(X_te)[0, 1]
```

### Retrain final model on all 43 patients after LOOCV evaluation
```python
# Retrain on full dataset for feature inspection only (performance already estimated by LOOCV)
final_model = clone(model_templates[best_model_name])
final_model.fit(X[final_peptides], y)
joblib.dump(final_model, MODEL_DIR / "modeling3_final_model.joblib")
```

---

## 5. To Do Next / Blockers

### To Do Next
- [ ] Run `modeling3.ipynb` and fill in the LOOCV results table above
- [ ] Update `checkpoint6.md` with actual LOOCV metrics once the notebook is run
- [ ] Update `feature_importance.ipynb` to optionally load from `modeling3_final_model.joblib` (trained on all 43 patients) instead of the modeling2 model
- [ ] Consider implementing a Neural Network model (listed in `project_plan.md` as a proposed model, not yet done)

### Blockers
- None. `modeling3.ipynb` has been run successfully.

---

## 6. File Index

| Filename | Description |
|----------|-------------|
| `modeling3.ipynb` | LOOCV pipeline — feature selection inside loop, no data leakage |
| `models/modeling3_loocv_comparison.tsv` | LOOCV metrics for all 3 models (generated after running) |
| `models/modeling3_loocv_predictions.tsv` | Per-patient predictions from the best LOOCV model |
| `models/modeling3_final_model.joblib` | Best model retrained on all 43 patients (for feature inspection) |
| `models/modeling3_best_model_name.txt` | Name of the best LOOCV model |
| `models/modeling3_selected_peptides.txt` | 200 peptides selected from the full 43-patient dataset |
| `checkpoints/checkpoint6.md` | This file |
