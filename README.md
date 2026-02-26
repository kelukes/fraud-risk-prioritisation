# fraud-risk-prioritisation
Binary fraud-risk prioritisation for motor insurance claims.  
We train on earlier years and evaluate on a future year (**time split**) and report:
- PR-AUC (Average Precision) as the primary ranking metric
- Top-K prioritisation metrics (Precision@K, Recall@K, Lift@K) to reflect limited investigation capacity

## Data access

This repository does not include the raw dataset file.

## Repository structure
- `notebooks/`
  - `01_EDA_claim.ipynb` — EDA + preprocessing decisions
  - `02_models_and_evaluation.ipynb` — modeling, tuning, Top-K evaluation, error analysis, SHAP
- `data/`
  - `data_source.md` — dataset link and download notes
  - `data_dictionary.md` — empirical feature dictionary (no official schema provided)
- `reports/`
  - `01_problem_framing.md`
  - `02_stakeholder_report.pdf`


## How to reproduce
1. Download the dataset CSV from Kaggle.
2. Place it into `data/`.
3. Set the expected filename inside the notebook (e.g., `insurance_claims.csv`).
4. Run the notebooks in order:
   - `notebooks/01_EDA_claim.ipynb`
   - `notebooks/02_models_and_evaluation.ipynb`

## Notes
- Preprocessing and feature engineering are fully documented in the notebooks.
- The model is designed for human-in-the-loop prioritisation, not automatic claim denial.
