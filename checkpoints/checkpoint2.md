# Checkpoint 2 — Data Exploration & Visualization
**Date:** 2026-04-22
**Project:** Severe vs Non-severe COVID-19 Classifier
**Dataset:** Proteomic and Metabolomic Characterization of COVID-19 Patient Sera (MSV000085507)

---

## 1. Goal
Understand what the data looks like before building any models — check data quality through missing value analysis, and visualize intensity patterns to confirm there is separable signal between Severe and Non-severe COVID-19 patients.

---

## 2. Summary
We reloaded the cleaned dataset from Checkpoint 1 (`variants_filtered.tsv`, 43 patients × 101,461 peptides) and ran four exploratory analyses. First, we quantified missing values both per patient and per peptide to understand how sparse the data is. Second, we plotted a histogram of the single most complete peptide, comparing its intensity distribution between Severe and Non-severe patients. Third, we made a scatter plot using the two most complete peptides to see whether two features alone can separate the groups (they cannot — which is expected and motivates the modelling steps). Finally, we built a heatmap of the top 50 highest-variance peptides (present in ≥80% of patients), log-transforming and z-scoring the intensities so all peptides are on the same scale, and sorting patients by condition to visually check for group structure.

---

## 3. Key Results
| Property | Value |
|----------|-------|
| Dataset reloaded | 43 patients × 101,461 peptides |
| Peptides selected for heatmap | Top 50 by variance, present in ≥80% of patients |
| Intensity transformation | log1p → z-score per peptide |
| Two-peptide scatter plot | Groups overlap — single peptides are not sufficient to separate classes |
| Heatmap | Patients sorted by condition; colour blocks indicate whether group signal exists |

---

## 4. Important Code

### Check missing values per patient and per peptide
```python
# Count NaNs across every patient (row) and every peptide (column)
na_per_patient = X.isna().sum(axis=1).sort_values(ascending=False)
na_per_peptide = X.isna().sum(axis=0)
print(f"Missing: {X.isna().sum().sum():,} / {X.size:,}  ({100*X.isna().sum().sum()/X.size:.1f}%)")
```

### Histogram — intensity distribution for one peptide
```python
# Compare intensity of the most complete peptide between the two groups
best_peptide = na_per_peptide.idxmin()
severe     = X.loc[condition == "Severe-COVID-19",     best_peptide].dropna()
non_severe = X.loc[condition == "Non-severe-COVID-19", best_peptide].dropna()

ax.hist(non_severe, bins=15, alpha=0.6, label="Non-severe", color="steelblue")
ax.hist(severe,     bins=15, alpha=0.6, label="Severe",     color="tomato")
```

### Heatmap — top 50 peptides, z-scored, patients sorted by condition
```python
# Select high-variance peptides, log-transform then z-score, plot as heatmap
min_present  = int(0.8 * len(X))
well_covered = X.columns[X.notna().sum() >= min_present]
top50        = X[well_covered].var().nlargest(50).index
patient_order = condition.sort_values().index

heatmap_log = np.log1p(X.loc[patient_order, top50])
heatmap_z   = (heatmap_log - heatmap_log.mean()) / heatmap_log.std()
ax.imshow(heatmap_z.T, aspect="auto", cmap="RdBu_r", vmin=-2, vmax=2)
```

---

## 5. To Do Next / Blockers

### To Do Next
- [ ] Run a t-test on every peptide to find which ones differ most between Severe and Non-severe
- [ ] Run a Mann-Whitney test as a safer non-parametric alternative (small n=43)
- [ ] Rank peptides by p-value and filter to those passing a significance threshold (e.g. p < 0.05)

### Blockers
- Missing value strategy is not yet decided — need to choose between dropping sparse peptides vs. imputation before running statistical tests and building models.

---

## 6. File Index
| Filename | Description |
|----------|-------------|
| `variants_filtered.tsv` | Input for this checkpoint — 43 patients × 101,461 peptides (from Checkpoint 1) |
| `variants_tmt.ipynb` | Notebook containing all Checkpoint 2 code and plots |
| `checkpoints/checkpoint2.md` | This file |
