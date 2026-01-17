# Early Infection Prediction in the NICU (MIMIC-III)

This repository documents the end-to-end workflow for predicting neonatal ICU (NICU) infection within the first 30/120 minutes after admission using MIMIC-III v1.4. It covers data extraction, feature aggregation, missing-value handling, model selection, tuning, and interpretation.

## Repository Structure
- `code/00_data_gathering/`: SQL scripts to extract NICU stays from `chartevents` and `labevents` (first NICU stay only, alive/in-unit beyond 120 minutes, excludes repeat NICU admissions; 120-minute window after admission).
- `code/01_EDA/`: Basic exploration and missing-rate checks.
- `code/02_data_cleaning/`: Missingness analysis and MICE imputation experiments for 30/120-minute windows.
- `code/03_model_selection/`: Feature selection and model search with CatBoost and Gradient Boosting (30/120-minute windows).
- `code/04_fine_turing_feature_selection/`: CatBoost fine-tuning and feature-importance analysis.
- `code/05_final_conclusion/`: Final grid searches and SHAP interpretation (`data550_*.ipynb` series).
- `data/`: Processed feature tables and model outputs (details below).

## Data Notes
- Source: MIMIC-III v1.4; extraction SQL in `code/00_data_gathering/`.
- Filters: first NICU admission only; excludes patients who die/discharge within 120 minutes; excludes subjects with multiple NICU stays; measurements limited to the first 120 minutes after admission.
- Feature construction: vitals and labs aggregated by min/max/mean over 30- and 120-minute windows; label column is `is_infected` (0/1).
- Key files (each has 6,539 rows and 86 columns including the label):
  - `data/nicu_30.csv`, `data/nicu_120.csv`: 30/120-minute aggregated features.
  - `data/nicu_30_cb_data.csv`, `data/nicu_120_cb_data.csv`: Top-15 CatBoost grid-search results with CV score, precision, recall, F1, accuracy.
  - `data/nicu_30_gb_data.csv`, `data/nicu_120_gb_data.csv`: Corresponding Gradient Boosting results.
  - Other intermediates: `nicu_chartevents_filtered.csv`, `nicu_labevents_filtered.csv` (raw SQL exports), `nicu_*_mean.csv`, `summary_stats.csv`, etc.
- Note: Some notebooks reference `../data/final/`; current data live in `data/` at the repo root. Adjust `file_path` or place the files under `data/final/` if needed.

## Environment & Dependencies
- Python 3.9+.
- Key libraries: pandas, numpy, scikit-learn, catboost, shap, matplotlib / seaborn, jupyterlab / notebook.
- Quick install:
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate  # On Windows: .venv\\Scripts\\activate
  pip install pandas numpy scikit-learn catboost shap matplotlib seaborn jupyterlab
  ```

## How to Run
1. Obtain authorized MIMIC-III v1.4 access. Run the SQL (MySQL syntax) in `code/00_data_gathering/` to export `nicu_chartevents_filtered.csv` and `nicu_labevents_filtered.csv`.
2. Aggregate the raw exports into 30/120-minute feature tables and place them under `data/` (or the path referenced by your notebooks).
3. Exploration/preprocessing: see `code/01_EDA/` and `code/02_data_cleaning/` for missing-rate and imputation experiments.
4. Model training and interpretation:
   - CatBoost: `code/05_final_conclusion/data550_cb30.ipynb`, `data550_cb120.ipynb` (MICE imputation + StratifiedKFold + GridSearchCV + SHAP).
   - Gradient Boosting: `code/05_final_conclusion/data550_gb30.ipynb`, `data550_gb120.ipynb`.
   - Running these notebooks produces/updates the corresponding result CSVs (e.g., `data/nicu_120_cb_data.csv`).
5. Other utilities: `demographic.ipynb`, `data550_data_generate.ipynb` for additional statistics or data generation.

## Current CV Snapshot
- CatBoost 120-minute window: accuracy ≈ 0.832 (`data/nicu_120_cb_data.csv`, rank 1).
- CatBoost 30-minute window: accuracy ≈ 0.849 (`data/nicu_30_cb_data.csv`, rank 1).
- Gradient Boosting 120-minute window: accuracy ≈ 0.829 (`data/nicu_120_gb_data.csv`, rank 4).
- Gradient Boosting 30-minute window: accuracy ≈ 0.853 (`data/nicu_30_gb_data.csv`, rank 1).

## Future Improvements
- Explore additional time windows (e.g., 60 minutes) and imbalance handling (class weights, sampling).
- Compare against clinical rules or other models (XGBoost, LightGBM, Logistic Regression).
- Add external or temporal validation to check generalization.

