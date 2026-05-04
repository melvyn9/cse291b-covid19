# Checkpoint 1 — Data Loading & Cleaning
**Date:** 2026-04-22
**Project:** Severe vs Non-severe COVID-19 Classifier
**Dataset:** Proteomic and Metabolomic Characterization of COVID-19 Patient Sera (MSV000085507)

---

## 1. Goal
Load the raw mass spectrometry dataset, remove non-patient columns, clean missing values, and produce a filtered dataset containing only the 43 patients relevant to the binary classification task (Severe vs Non-severe COVID-19).

---

## 2. Summary
We loaded the raw 414MB TSV file (`variants_tmt.tsv`) which contained 101,461 peptide variants measured across 268 columns. Of those 268 columns, 92 were patient intensity measurements — but 2 of them were lab placeholders (an Empty channel and a Norm reference channel), not real patients. We removed those, leaving 90 real patients across 4 disease groups. We then replaced all zero intensity values with NaN, since in mass spectrometry a zero means "not detected" rather than a true zero amount. We transposed the matrix so that each row represents a patient and each column represents a peptide, then filtered down to only the 43 patients relevant to our task: 18 Severe and 25 Non-severe COVID-19 patients. The result was saved as `variants_filtered.tsv`.

---

## 3. Key Results
| Property | Value |
|----------|-------|
| Raw file size | 414 MB |
| Raw shape | 101,461 peptides × 268 columns |
| Patient intensity columns found | 92 |
| Non-patient columns removed | 2 (Empty, Norm) |
| Real patients (all 4 classes) | 90 |
| Final shape after filtering | 43 patients × 101,461 peptides |
| Severe-COVID-19 | 18 patients |
| Non-severe-COVID-19 | 25 patients |

---

## 4. Important Code

### Find and filter to real patient columns
```python
# Identify all intensity columns, then remove lab placeholder channels
intensity_cols = [c for c in variants.columns if 'intensity_for_peptide_variant' in c]

real_patient_cols = [c for c in intensity_cols
                     if not c.startswith('_dyn_#Empty')
                     and not c.startswith('_dyn_#Norm')]
```

### Clean, transpose, and add condition labels
```python
# Replace 0s with NaN, make Peptide the index, transpose to (patients × peptides)
variants_processed = variants[['Peptide'] + real_patient_cols]
variants_processed = variants_processed.replace(0.0, float('nan'))
variants_processed = variants_processed.set_index('Peptide')
variants_processed = variants_processed.T

# Shorten row names to "Class.PatientID" and add a Condition column
variants_processed.index = variants_processed.index.map(
    lambda x: '.'.join(x.replace('_dyn_#', '').split('.')[:2])
)
variants_processed['Condition'] = variants_processed.index.map(
    lambda x: x.split('.')[0]
)
```

### Filter to Severe vs Non-severe and save
```python
# Keep only the 2 classes relevant to the classification task
variants_filtered = variants_processed[
    (variants_processed['Condition'] == 'Severe-COVID-19') |
    (variants_processed['Condition'] == 'Non-severe-COVID-19')
]

variants_filtered.to_csv("variants_filtered.tsv", sep="\t")
```

---

## 5. To Do Next / Blockers

### To Do Next
- [ ] Check missing values — how many NaNs per peptide and per patient?
- [ ] Visualize intensity distributions (histogram, scatter plot, heatmap)
- [ ] Confirm there is separable signal between Severe and Non-severe groups

### Blockers
None.

---

## 6. File Index
| Filename | Description |
|----------|-------------|
| `variants_tmt.tsv` | Raw input file — 101,461 peptide variants × 268 columns (not committed, too large) |
| `variants_filtered.tsv` | Cleaned output — 43 patients × 101,461 peptides, Severe and Non-severe only |
| `checkpoints/checkpoint1.md` | This file |
