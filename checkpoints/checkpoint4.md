# Checkpoint 4 — Building the Classifier
**Date:** 2026-05-09
**Project:** Severe vs Non-severe COVID-19 Classifier
**Dataset:** Variants TMT / `checkpoint4_modeling_input_top200.tsv`

---

## 1. Goal
Train a first classifier to predict Severe vs Non-severe COVID-19 using the top 200 statistically ranked peptides selected in Checkpoint 3.

---

## 2. Summary
We moved from statistical feature selection into modeling. The notebook loads the Checkpoint 3 modeling input file, encodes Severe patients as 1 and Non-severe patients as 0, splits the 43 patients into training, validation, and test sets, handles missing peptide intensities with median imputation, and trains three baseline models. The validation set is used to choose the best baseline model, while the test set is saved for final evaluation in Checkpoint 5.

---

## 3. Key Results

- Modeling input: 43 patients × 200 peptide features, plus `Condition`
- Label encoding: Non-severe-COVID-19 = 0; Severe-COVID-19 = 1
- Train split: 25 patients
  - 15 Non-severe
  - 10 Severe
- Validation split: 9 patients
  - 5 Non-severe
  - 4 Severe
- Test split: 9 patients
  - 5 Non-severe
  - 4 Severe
- Models trained:
  - Logistic Regression with L2 regularization
  - Logistic Regression with L1 regularization
  - Random Forest
- Model selection metric: validation F1 score for the Severe class
- Final test evaluation is intentionally held for Checkpoint 5

---

## 4. Important Code

### Load the modeling input
# This loads the top 200 peptide table created in Checkpoint 3.
```python
INPUT_FILE = Path("checkpoint4_modeling_input_top200.tsv")
df = pd.read_csv(INPUT_FILE, sep="\t", index_col=0)
condition = df["Condition"]
X = df.drop(columns=["Condition"])
```

### Encode labels
# This converts class names into numeric labels for modeling.
```python
LABEL_MAP = {
    "Non-severe-COVID-19": 0,
    "Severe-COVID-19": 1,
}
y = condition.map(LABEL_MAP)
```

### Split the data
# This creates train, validation, and test sets while preserving class balance.
```python
X_trainval, X_test, y_trainval, y_test = train_test_split(
    X, y,
    test_size=0.20,
    stratify=y,
    random_state=42,
)

X_train, X_val, y_train, y_val = train_test_split(
    X_trainval, y_trainval,
    test_size=0.25,
    stratify=y_trainval,
    random_state=42,
)
```

### Build a baseline logistic regression model
# This imputes missing values, scales features, and trains a balanced logistic regression model.
```python
logistic_regression_l2 = Pipeline(steps=[
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler()),
    ("model", LogisticRegression(
        penalty="l2",
        C=1.0,
        class_weight="balanced",
        max_iter=5000,
        random_state=42,
    )),
])
```

### Save Checkpoint 4 outputs
# This saves the split data, validation results, and selected model for the next checkpoint.
```python
train_df.to_csv("checkpoint4_train_top200.tsv", sep="\t")
val_df.to_csv("checkpoint4_validation_top200.tsv", sep="\t")
test_df.to_csv("checkpoint4_test_top200.tsv", sep="\t")
model_comparison.to_csv("checkpoint4_model_comparison.tsv", sep="\t", index=False)
joblib.dump(best_model, "checkpoint4_best_model.joblib")
```

---

## 5. To Do Next / Blockers

### To Do Next
- [ ] Run the Checkpoint 4 notebook after `checkpoint4_modeling_input_top200.tsv` exists in the working folder
- [ ] Review `checkpoint4_model_comparison.tsv` to confirm which model performed best on validation
- [ ] Use the saved test set and best model for Checkpoint 5 evaluation

### Blockers
The notebook requires `checkpoint4_modeling_input_top200.tsv` from Checkpoint 3. If that file is missing, rerun the final Checkpoint 3 save cell first.

---

## 6. File Index

| Filename | Description |
|----------|-------------|
| `modeling.ipynb` | Notebook for Checkpoint 4 model training and validation |
| `checkpoint4.md` | Plain-English summary of Checkpoint 4 |
| `checkpoint4_modeling_input_top200.tsv` | Input file from Checkpoint 3 containing Condition plus 200 selected peptides |
| `checkpoint4_train_top200.tsv` | Training split saved by the notebook |
| `checkpoint4_validation_top200.tsv` | Validation split saved by the notebook |
| `checkpoint4_test_top200.tsv` | Held-out test split saved for Checkpoint 5 |
| `checkpoint4_model_comparison.tsv` | Validation metrics for each baseline model |
| `checkpoint4_validation_predictions.tsv` | Validation predictions from the selected model |
| `checkpoint4_best_model.joblib` | Saved selected baseline model |