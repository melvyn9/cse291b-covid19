# Checkpoint 5 — Evaluation & Corrected Pipeline
**Date:** 2026-04-22
**Project:** Severe vs Non-severe COVID-19 Classifier
**Dataset:** Proteomic and Metabolomic Characterization of COVID-19 Patient Sera (MSV000085507)

---

## 1. Goal
Evaluate the trained classifier on the held-out test set, identify a data leakage issue in the original pipeline, fix it, and re-run the full modeling workflow with a corrected feature selection strategy.

---

## 2. Summary
We first ran test set evaluation on the best model from Checkpoint 4 (Logistic Regression L2), which produced 100% accuracy across all metrics. On investigation, we found this was caused by data leakage: the statistical feature selection step (t-test and Mann-Whitney) had been run on all 43 patients before the train/test split, meaning the test patients influenced which 200 peptides were chosen as features.

We fixed this in `modeling2.ipynb` by moving the train/validation/test split to before any feature selection, then running all statistical tests exclusively on the 25 training patients. The same 200 selected peptides were applied to validation and test without re-selection. All imputation and scaling continued to use training-set statistics only.

The corrected pipeline selected Logistic Regression L1 as the best model (versus L2 in the leaky pipeline). Validation and test accuracy both dropped from 100% to 88.9%, with precision of 0.80 and perfect recall of 1.0 on the Severe class — meaning the model caught all Severe patients but produced one false positive (a Non-severe patient misclassified as Severe).

---

## 3. Key Results

| Metric | Leaky pipeline (modeling.ipynb) | Corrected pipeline (modeling2.ipynb) |
|--------|--------------------------------|--------------------------------------|
| Best model | Logistic Regression L2 | Logistic Regression L1 |
| Validation accuracy | 1.000 | 0.889 |
| Validation F1 (Severe) | 1.000 | 0.889 |
| Validation ROC-AUC | 1.000 | 0.950 |
| **Test accuracy** | — | **0.889** |
| **Test F1 (Severe)** | — | **0.889** |
| **Test ROC-AUC** | — | **0.950** |
| **Test recall (Severe)** | — | **1.000** (all 4 Severe patients caught) |
| **Test precision (Severe)** | — | **0.800** (1 false positive) |

---

## 4. Important Code

### Corrected preprocessing pipeline
```python
# Step 1 — Remove non-patient columns
# 101,461 × 268 → 101,461 × 90
intensity_cols    = [c for c in variants.columns if 'intensity_for_peptide_variant' in c]
real_patient_cols = [c for c in intensity_cols
                     if not c.startswith('_dyn_#Empty')
                     and not c.startswith('_dyn_#Norm')]

# Step 2 — Replace zeros with NaN
# Shape unchanged: 101,461 × 90  (59% of values now NaN)
variants_processed = variants_processed.replace(0.0, float('nan'))

# Step 3 — Filter to Severe/Non-severe, transpose
# 101,461 × 90 → 43 × 101,461
variants_filtered = variants_processed[
    variants_processed['Condition'].isin(['Severe-COVID-19', 'Non-severe-COVID-19'])
].T
```

### Split BEFORE feature selection
```python
# Step 4 — Train/val/test split first (key fix)
# Train: 25 × 101,461 | Val: 9 × 101,461 | Test: 9 × 101,461
X_trainval, X_test, y_trainval, y_test = train_test_split(
    X, y, test_size=0.20, stratify=y, random_state=42
)
X_train, X_val, y_train, y_val = train_test_split(
    X_trainval, y_trainval, test_size=0.25, stratify=y_trainval, random_state=42
)
```

### Feature selection on training set only
```python
# Step 5 — Statistical tests on X_train only
# 25 × 101,461 → 25 × 200 (and same 200 applied to val/test)
for peptide in X_train.columns:
    severe_vals    = X_train.loc[condition_train == "Severe-COVID-19",     peptide].dropna()
    nonsevere_vals = X_train.loc[condition_train == "Non-severe-COVID-19", peptide].dropna()
    if len(severe_vals) < 3 or len(nonsevere_vals) < 3:
        continue
    _, t_p  = ttest_ind(severe_vals, nonsevere_vals, equal_var=False)
    _, mw_p = mannwhitneyu(severe_vals, nonsevere_vals, alternative="two-sided")

selected_peptides = stats_train.dropna(subset=["mannwhitney_pvalue"]).head(200)["Peptide"].tolist()
X_train_sel = X_train[selected_peptides]
X_val_sel   = X_val[selected_peptides]    # same columns, no re-selection
X_test_sel  = X_test[selected_peptides]   # same columns, no re-selection
```

---

## 5. To Do Next / Blockers

### To Do Next
- [ ] Inspect the top peptide coefficients from the corrected Logistic Regression L1 model to identify the most important features
- [ ] Consider cross-validation (e.g. leave-one-out) to get a more stable performance estimate given n=43
- [ ] Explore whether a different top-N cutoff (e.g. top 50, top 500) changes corrected test performance

### Blockers
- With only 9 test patients, a single misclassification moves accuracy by ~11%. Results should be interpreted with caution — a larger cohort would be needed to draw reliable conclusions about true generalisation performance.

---

## 6. File Index

| Filename | Description |
|----------|-------------|
| `variants_filtered.tsv` | Cleaned input — 43 patients × 101,461 peptides (from Checkpoint 1) |
| `modeling.ipynb` | Original (leaky) pipeline — feature selection before split |
| `modeling2.ipynb` | Corrected pipeline — feature selection on training set only |
| `models/checkpoint4_best_model.joblib` | Best model from leaky pipeline (for reference) |
| `checkpoints/checkpoint5.md` | This file |
