# Checkpoint 3 — Statistical Analysis
**Date:** 2026-05-09
**Project:** Severe vs Non-severe COVID-19 Classifier  
**Dataset:** Proteomic and Metabolomic Characterization of COVID-19 Patient Sera — variants TMT data

---

## 1. Goal
Find which peptide measurements differ most between Severe and Non-severe COVID-19 patients, then save one clear feature-limited TSV to use as the input dataset for Checkpoint 4 modeling.

---

## 2. Summary
We compared each peptide between the 18 Severe and 25 Non-severe patients using Welch's t-test and the Mann-Whitney U test. No peptides passed FDR < 0.05, which is expected with about 101,000 peptide tests and only 43 patients. Because `significant_fdr` is empty, it should not be used for modeling. Instead, Checkpoint 3 now saves `checkpoint4_modeling_input_top200.tsv`, which contains `Condition` plus the top 200 peptides ranked by Mann-Whitney p-value, then t-test p-value.

---

## 3. Key Results

| Item | Result |
|------|--------|
| Input dataset | `variants_filtered.tsv` |
| Dataset shape | 43 patients × 101,461 peptides + `Condition` |
| Severe patients | 18 |
| Non-severe patients | 25 |
| Statistical tests | Welch's t-test and Mann-Whitney U test |
| Primary ranking | Mann-Whitney p-value, then t-test p-value |
| FDR result | No peptides passed FDR < 0.05 |
| Modeling input selected | Top 200 ranked peptides |
| File to use in Checkpoint 4 | `checkpoint4_modeling_input_top200.tsv` |

---

## 4. Important Code

### Rank peptides statistically
# Compares Severe and Non-severe patients for each peptide
```python
from scipy.stats import ttest_ind, mannwhitneyu

results = []

for peptide in X.columns:
    severe_vals = X.loc[condition == "Severe-COVID-19", peptide].dropna()
    nonsevere_vals = X.loc[condition == "Non-severe-COVID-19", peptide].dropna()

    if len(severe_vals) < 3 or len(nonsevere_vals) < 3:
        continue

    t_stat, t_p = ttest_ind(
        severe_vals,
        nonsevere_vals,
        equal_var=False,
        nan_policy="omit"
    )

    u_stat, mw_p = mannwhitneyu(
        severe_vals,
        nonsevere_vals,
        alternative="two-sided"
    )

    results.append({
        "Peptide": peptide,
        "n_severe": len(severe_vals),
        "n_nonsevere": len(nonsevere_vals),
        "mean_severe": severe_vals.mean(),
        "mean_nonsevere": nonsevere_vals.mean(),
        "log2_fold_change": np.log2((severe_vals.mean() + 1) / (nonsevere_vals.mean() + 1)),
        "t_pvalue": t_p,
        "mannwhitney_pvalue": mw_p
    })

stats_df = pd.DataFrame(results)
stats_ranked = stats_df.sort_values(
    ["mannwhitney_pvalue", "t_pvalue"],
    na_position="last"
).reset_index(drop=True)
```

### Create the Checkpoint 4 input TSV
# Saves the exact file to load at the start of Checkpoint 4
```python
N_MODELING_PEPTIDES = 200

modeling_feature_stats = (
    stats_ranked
    .dropna(subset=["mannwhitney_pvalue"])
    .head(N_MODELING_PEPTIDES)
    .copy()
)
selected_peptides = modeling_feature_stats["Peptide"].tolist()

checkpoint4_modeling_input_top200 = pd.concat(
    [condition.rename("Condition"), X[selected_peptides]],
    axis=1
)

checkpoint4_modeling_input_top200.to_csv(
    "checkpoint4_modeling_input_top200.tsv",
    sep="\t"
)

modeling_feature_stats.to_csv(
    "checkpoint4_selected_peptides_top200.tsv",
    sep="\t",
    index=False
)
```

---

## 5. To Do Next / Blockers

### To Do Next
- [ ] In Checkpoint 4, load `checkpoint4_modeling_input_top200.tsv` directly.
- [ ] Split patients into training and test sets before imputing missing values.
- [ ] Impute missing peptide values using training-set medians.
- [ ] Train a simple baseline classifier, starting with logistic regression.
- [ ] Optionally compare top 50, top 100, top 200, and top 500 peptide sets later.

### Blockers
None. The empty FDR table is expected and should not block modeling.

---

## 6. File Index

| Filename | Description |
|----------|-------------|
| `variants_filtered.tsv` | Input for this checkpoint — 43 patients × 101,461 peptides (from Checkpoint 1) |
| `peptide_statistical_tests.tsv` | Full ranked statistical results for all tested peptides. |
| `significant_peptides_p005.tsv` | Peptides passing raw p-value < 0.05. Useful for inspection, but less strict than FDR. |
| `significant_peptides_fdr005.tsv` | Peptides passing FDR < 0.05. This is empty for this dataset. |
| `top50_statistical_peptides.tsv` | The 50 strongest individual peptide signals by statistical ranking. Useful for inspection, not the main modeling input. |
| `checkpoint4_selected_peptides_top200.tsv` | Statistical details for the 200 peptide features selected for modeling. |
| `checkpoint4_modeling_input_top200.tsv` | Main input file for Checkpoint 4: 43 patients × `Condition` + 200 selected peptide intensity columns. |
| `variants_tmt.ipynb` | Updated notebook that performs Checkpoint 3 and writes the Checkpoint 4 input TSV. |
