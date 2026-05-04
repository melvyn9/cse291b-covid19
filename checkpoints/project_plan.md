Project Plan — Severe vs Non-severe COVID-19 Classifier

✅ Checkpoint 1 — Data Loading & Cleaning (Done)

Loaded raw dataset
Cleaned and filtered to 43 patients × 101,461 peptides
Saved variants_filtered.tsv


📊 Checkpoint 2 — Data Exploration & Visualization
Goal: Understand what the data looks like before building any models.

Reload variants_filtered.tsv directly (no need to reprocess)
Check missing values — how many NaNs are there per peptide and per patient?
Histogram — pick one peptide and plot its intensity distribution for Severe vs Non-severe
Scatter plot — compare two peptides at once to see if they separate the two groups
Heatmap — visualize intensity patterns across all patients for the top peptides


📐 Checkpoint 3 — Statistical Analysis
Goal: Find which peptides are most different between Severe and Non-severe patients.

T-test — for each peptide, test if the means differ between the two groups
Mann-Whitney test — a safer version that doesn't assume normal distribution
Rank peptides by p-value — find the most statistically significant ones
Filter — keep only peptides that pass a significance threshold (e.g. p-value < 0.05)


🤖 Checkpoint 4 — Building the Classifier
Goal: Train a model to predict Severe vs Non-severe from peptide intensities.

Handle missing values — decide how to fill or drop NaNs for the model
Split data into train, validation and test sets
Choose a model — options include:

Logistic Regression (simple, good starting point)
Random Forest (handles missing data well)
Neural Network (more complex, like the starter code)


Train the model on the training set
Tune using the validation set


📈 Checkpoint 5 — Evaluation
Goal: Measure how well the model performs.

Accuracy — what % of patients did we classify correctly?
Precision & Recall — are we better at catching Severe cases or Non-severe?
F1 Score — a balance between precision and recall
Confusion matrix — visualize correct vs incorrect predictions
Precision-Recall curve — see how the model performs at different thresholds