# fraud-risk-prioritisation
A binary classification problem in the context of motor insurance claims using CatBoost evaluated by time split with with PR-AUC and top_K 

# Data access

This repository does not include the raw dataset file.

## Source
Dataset: "Vehicle Insurance Claim Fraud Detection" (Kaggle)
https://www.kaggle.com/datasets/shivamb/vehicle-claim-fraud-detection/data

## How to reproduce
1. Download the dataset CSV from Kaggle.
2. Place the file into this folder: `data/`
3. Expected filename: `insurance_claims.csv`  (change to your real filename)
4. Run the notebook: `notebooks/classification_capstone.ipynb`

## Notes
- The notebook applies preprocessing and feature engineering steps documented in the notebook.
- ID-like fields (e.g., PolicyNumber) are dropped before modeling.
