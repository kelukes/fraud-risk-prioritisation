# Problem Definition, Dataset, and Evaluation Plan (Updated)

## 1. Problem Definition & Context

### 1.1 Clear classification problem statement

We address a binary classification problem in the context of motor insurance claims.  
Each claim is described by policy, vehicle, driver, and incident characteristics and labeled as either:

- `FraudFound_P = 1` — fraudulent claim (positive class)  
- `FraudFound_P = 0` — legitimate claim (negative class)

**Goal.** Build a model that, given a new claim’s attributes available at reporting time, outputs a **fraud-risk score** used to **rank** claims for human investigation.  
In this project, the model is treated primarily as a **prioritization tool (Top-K queue)** rather than a fixed-threshold auto-decision system. It is not intended to automatically deny claims.



### 1.2 Hypothesis: what the model will predict and why it matters

Our working hypothesis is that fraudulent claims follow different patterns than legitimate ones across combinations of features, such as:

- **Timing / process signals:** `Month`, `DayOfWeek`, `Days_Policy_Accident`, `Days_Policy_Claim`, claim delay proxies  
- **Policy and vehicle structure:** `PolicyType`, `CoverageType`, `VehicleCategory`, `VehiclePrice`, `AgeOfVehicle`, `Make`, `Deductible`, `DriverRating`  
- **Contextual signals:** `AccidentArea`, `PoliceReportFiled`, `WitnessPresent`, `PastNumberOfClaims`, `AddressChange_Claim`, `NumberOfSuppliments`

If these patterns are learnable, a supervised model can assign higher fraud scores to claims whose feature combinations are more typical for historical fraud than for legitimate cases.

**Why it matters:** False negatives (missed fraud) increase financial loss and indirectly raise premiums for honest customers. False positives (legitimate claims flagged) overload investigators and can harm customer trust. Because investigation resources are limited, the practical objective is to **concentrate fraud into a limited investigation queue** rather than to perfectly classify every case.



### 1.3 Societal / sustainability relevance and stakeholders

Although this project is not about environmental impact, it relates to social and economic sustainability:

- Persistent insurance fraud makes the system less sustainable: honest customers subsidize fraud through higher premiums or reduced coverage.
- More accurate fraud prioritization reduces leakage and supports fairer, more stable pricing.
- Transparent and well-governed decision support can increase trust by making investigations more consistent and evidence-based.

Key stakeholders include:

- **Insurance management** (loss reduction, exposure, long-term sustainability)
- **Fraud investigation / claims teams** (prioritized and interpretable queues, manageable workload)
- **Honest policyholders** (affected by undetected fraud and by unfair suspicion / friction)
- **Regulators / consumer protection bodies** (fairness, transparency, and governance)

In this project, the model is explicitly treated as human-in-the-loop decision support, not an automatic rejection tool.


## 2. Dataset Selection & Description

### 2.1 Rationale for chosen dataset and target variable

This project uses the Kaggle dataset commonly known as “Vehicle Insurance Claim Fraud Detection” with target variable `FraudFound_P`.

Reasons for selection:

- Clear binary target (`FraudFound_P`)
- Realistic **class imbalance** (≈6% fraud), motivating evaluation beyond accuracy;
- Structured tabular features suitable for classical ML models;
- Enough rows/features to explore baseline models, ablations, imbalance handling, and explainability

We treat `FraudFound_P` as an insurer-provided fraud label. Due to limited documentation, we cannot fully confirm whether it reflects confirmed fraud vs flagged/suspected fraud, so results are interpreted with this uncertainty in mind.


### 2.2 Data source, access, and licensing / ethical considerations

- **Source:** Kaggle dataset circulated as an educational benchmark. It is often described as originating from an Oracle/insurance demonstration database, but the original collection and labeling details are not documented.
- **Access:** downloaded CSV from Kaggle and used locally in the notebook.
- **Anonymization:** no direct personal identifiers (names/addresses/contacts). Some ID-like codes may exist and are treated as technical identifiers (and excluded from modeling).

Ethical caveats:

- Labels reflect historical investigative decisions and internal rules and may embed bias/inconsistency.
- Single-context dataset may not represent other markets/products.
- In real deployment, such a model should be used for prioritization with human oversight and must comply with data protection and anti-discrimination regulations.


### 2.3 Initial structural properties and concerns

- Dataset size: **15,419** claims.
- Time coverage: **three years (1994–1996)**:
- Target imbalance: `FraudFound_P = 1` is ≈ 6% overall.
- Features: mixed categorical + numeric. A small set of additional domain-based features is engineered during modeling.

Key concerns:

- **Temporal drift / policy drift:** performance and patterns can change across years.
- **Label bias:** target may reflect process decisions.
- **Imbalance:** accuracy is misleading; evaluation must focus on ranking and capacity-driven trade-offs.


## 3. Success Criteria & Evaluation Plan

### 3.1 Objective framing: ranking + investigation capacity (Top-K)

Fraud detection is framed as a ranking problem. The output is a score used to prioritize limited investigation capacity.

Therefore, success is defined by:
1) how well the model ranks fraud above non-fraud (**threshold-free ranking metrics**), and  
2) how effective it is under a realistic **Top-K investigation policy**.


### 3.2 Primary selection metric for model choice

**Primary metric:** **PR-AUC (Average Precision, computed with scikit-learn)** on out-of-time validation (1995).  
**Secondary metric:** ROC-AUC (ranking sanity check).

PR-AUC is chosen because the dataset is imbalanced; it is more informative than accuracy and more sensitive to fraud-class ranking quality.


### 3.3 Operational evaluation (business-facing metrics)

We evaluate operational value using **Top-K prioritization metrics** on the held-out test year:

For K ∈ {1%, 5%, 10%, 20%}:
- **Precision@K:** fraud density within the investigated queue
- **Recall@K:** fraction of all fraud cases captured by investigating top K%
- **Lift@K:** Precision@K divided by fraud prevalence (random baseline)
- **TP / FP / FN counts:** explicit workload and missed-fraud trade-offs

We do not set a universal recall target (e.g., 90%), because under a capacity-driven policy recall depends on K.


### 3.4 Fairness and robustness checks (sanity level)

We do not claim to “solve fairness,” but we include checks to surface issues:

- interpretability via SHAP to verify whether model relies mainly on **product/process** features vs sensitive attributes
- segment-level error analysis under Top-K to detect **coverage blind spots**
- explicit discussion of risks from FP/FN and mitigation via policy and human oversight


### 3.5 Evaluation procedure

**Temporal evaluation protocol:**
- Selection: train **1994** -> validate **1995**
- Final test: train **1994–1995** -> test **1996**
We use a temporal split to approximate deployment on future data.  
A random stratified split would mix years and can overestimate performance under temporal drift.

Workflow:
1) Baseline modeling (Logistic Regression) and ablations / imbalance strategies to understand signal and drift.
2) Stronger non-linear model: **CatBoost**, suitable for categorical-heavy tabular data.
3) Hyperparameter selection using **PR-AUC on Val 1995** (primary).
4) Final reporting on Test 1996:
   - PR curve (ranking quality)
   - Top-K tables and plots (capacity-driven utility)
   - error analysis (missed fraud under Top-20%)
   - SHAP explanations (global drivers + local examples)
   - limitations and ethical reflection


## Ethical Reflection

### Dataset limitations and potential biases
- Under-documented labeling process (`FraudFound_P` may encode investigator behavior/policy).
- Only three years available -> limited temporal generalization evidence.
- Imbalanced target and rare fraud pockets in specific segments.

### Risks of harm from misclassification
- **FP:** unnecessary investigations, customer friction, potential unfair suspicion.
- **FN:** missed fraud -> financial loss and repeated fraud.
- **Blind spots:** entire segments may be deprioritized under Top-K, risking systematic misses.

### Mitigation strategies
- Policy design: choose K based on capacity; consider stratified sampling or minimum segment coverage to reduce blind spots.
- Transparency: communicate score is a prioritization tool, not a fraud verdict.
- Monitoring/retraining: track drift and recalibrate workflow as patterns change.
- Human-in-the-loop: prioritize for review, do not auto-deny.

### Data growth / improvement directions
If generalization is limited or variance is high, improvements would likely come from:
- more labeled data for rare fraud pockets and under-covered segments
- better coverage of edge cases (segment diversity)
- richer process features available at claim time (to distinguish subtle fraud)
- investigation policies that ensure some sampling in low-risk segments to obtain labels and monitor drift