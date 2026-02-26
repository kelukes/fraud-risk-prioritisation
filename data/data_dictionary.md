# Vehicle Claim Fraud — Data Dictionary

## Documentation note

This dataset does not provide an official data dictionary.  
Therefore, the definitions below are **empirical**: we infer meanings from observed values (category labels, ranges, and naming conventions) and explicitly mark uncertain interpretations.

**Goal of this document:**
- align on what each feature likely represents,
- identify **ID-like** or **process-related** variables,
- support later decisions about leakage risk and preprocessing.



## Target definition (working hypothesis)

- **Target column:** `FraudFound_P` (int, {0,1})
- **Positive class:** `1` (fraud)

### Working interpretation
`FraudFound_P = 1` indicates a claim that was **labeled by the insurer as fraud found/flagged** based on historical decisions.  
Because there is no official documentation, we cannot confirm whether this label corresponds to **confirmed fraud** or **suspected/flagged fraud**.

## Feature dictionary

### Time & calendar features

| Feature | Type | Observed values / examples | Hypothesized meaning | When available | Notes / Assumption level |
|---|---|---|---|---|---|
| `Year` | int (3 values) | 1994, 1995, 1996 | Calendar year of the record (accident and/or claim year) | claim-time | **Medium**: dataset lacks official doc, but values look like years. |
| `Month` | categorical (12) | Jan…Dec | Month of accident (or event month) | claim-time | **Medium**: likely accident-time, paired with `WeekOfMonth`, `DayOfWeek`. |
| `WeekOfMonth` | int (1–5) | 1..5 | Week-of-month of accident (or event week index) | claim-time | **Medium**. |
| `DayOfWeek` | categorical (7) | Monday…Sunday | Day of week of accident (or event day) | claim-time | **Medium**. |
| `MonthClaimed` | categorical (13) | months (+ possibly extra token) | Month when claim was filed | claim-time | **Medium**: verify the 13th category token later. |
| `WeekOfMonthClaimed` | int (1–5) | 1..5 | Week-of-month when claim was filed | claim-time | **Medium**. |
| `DayOfWeekClaimed` | categorical (8) | weekdays (+ possibly extra token) | Day of week when claim was filed | claim-time | **Medium**: verify the 8th category token later. |


### Customer / policyholder attributes

| Feature | Type | Observed values / examples | Hypothesized meaning | When available | Notes / Assumption level |
|---|---|---|---|---|---|
| `Sex` | categorical (2) | Male / Female | Sex of policyholder | policy-time | **High**. Sensitive attribute (fairness note later). |
| `MaritalStatus` | categorical (4) | e.g., Single/Married/... | Marital status | policy-time | **Medium**: confirm exact labels later if needed. |
| `Age` | int (66 unique) | includes `0`, 20, 34, 71… | Age in years | policy-time | **Medium**: `0` is a coded for age < 16 which was investigated during EDA. |
| `AgeOfPolicyHolder` | ordinal categorical (9) | “31 to 35”, “over 65”, … | Age band of policyholder | policy-time | **High**: explicit bins. Treat as ordinal or categorical. |


### Vehicle attributes

| Feature | Type | Observed values / examples | Hypothesized meaning | When available | Notes / Assumption level |
|---|---|---|---|---|---|
| `Make` | categorical (19) | 19 manufacturers | Vehicle manufacturer (brand) | policy-time | **High**. |
| `VehicleCategory` | categorical (3) | 3 categories | Broad vehicle type/category | policy-time | **Medium**: print exact labels later if needed. |
| `VehiclePrice` | ordinal categorical (6) | “less than 20000”, “20000 to 29000”, … | Vehicle price band | policy-time | **High**: explicit bins. |
| `AgeOfVehicle` | ordinal categorical (8) | “new”, “2 years”, “more than 7”, … | Vehicle age band | policy-time | **High**: explicit bins. |



### Policy & coverage attributes

| Feature | Type | Observed values / examples | Hypothesized meaning | When available | Notes / Assumption level |
|---|---|---|---|---|---|
| `PolicyType` | categorical (9) | “Sedan - Collision”, “Sedan - Liability”, … | Combined indicator of vehicle group + coverage type | policy-time | **High**: clearly structured label. Candidate to split into two features later. |
| `BasePolicy` | categorical (3) | 3 categories | Base coverage class (e.g., Liability/Collision/All Perils) | policy-time | **Medium**: likely overlaps with `PolicyType`. |
| `Deductible` | int (4 values) | 4 numeric values | Deductible amount (franchise) | policy-time | **Medium**: numeric but low-cardinality. |
| `DriverRating` | int (4 values) | 1..4 | Driver risk/rating score | policy-time | **Medium**: numeric but ordinal. |
| `Days_Policy_Accident` | ordinal categorical (5) | “none”, “1 to 7”, “8 to 15”, “15 to 30”, “more than 30” | Age of policy (in days band) at time of accident | claim-time | **Medium**: name suggests policy age vs accident time; distribution is highly skewed to “more than 30”. |
| `Days_Policy_Claim` | ordinal categorical (4) | “none”, “8 to 15”, “15 to 30”, “more than 30” | Age of policy (in days band) at time of claim filing | claim-time | **Medium**: highly skewed to “more than 30”. |



### Claim / accident context & behavioral signals

| Feature | Type | Observed values / examples | Hypothesized meaning | When available | Notes / Assumption level |
|---|---|---|---|---|---|
| `AccidentArea` | categorical (2) | Urban / Rural | Accident location type | claim-time | **High**. |
| `Fault` | categorical (2) | “Policy Holder”, “Third Party” | Who was at fault | claim-time | **Medium**: likely assessed during claim; may be stated/derived. |
| `PoliceReportFiled` | categorical (2) | Yes / No | Whether police report was filed | claim-time | **Medium**: could be known at claim filing; confirm usage later. |
| `WitnessPresent` | categorical (2) | Yes / No | Whether witness was present | claim-time | **Medium**: could be known at claim filing. |
| `AgentType` | categorical (2) | External / Internal | Claim/policy handled by internal vs external agent | claim-time | **Medium**: rare “Internal” category; may reflect process. |
| `NumberOfSuppliments` | ordinal categorical (4) | “none”, “1 to 2”, “3 to 5”, “more than 5” | Number of supplemental documents/updates added to the claim | claim-handling | **Medium**: may be a process variable; ablation later depending on scoring time. Observed fraud rate does *not* increase with more supplements. |
| `PastNumberOfClaims` | ordinal categorical (4) | “none”, “1”, “2 to 4”, “more than 4” | Number of past claims by policyholder | policy-time | **Medium–High**: explicit bins. |
| `AddressChange_Claim` | ordinal categorical (5) | “no change”, “under 6 months”, “1 year”, … | Time since address change (relative to claim) | claim-time | **Medium**: very rare “under 6 months”. |
| `NumberOfCars` | ordinal categorical (5) | “1 vehicle”, “2 vehicles”, “3 to 4”, … | Number of cars owned/covered | policy-time | **High**: explicit bins; very rare high counts. |



### Identifiers / administrative fields

| Feature | Type | Observed values / examples | Hypothesized meaning | When available | Notes / Assumption level |
|---|---|---|---|---|---|
| `PolicyNumber` | int (15,420 unique) | 1..15420 | Policy identifier | technical | **High**: behaves like unique ID -> should be dropped for modeling. |
| `RepNumber` | int (16 unique) | 16 values | Representative/agent identifier | technical / process | **Medium**: not unique; may capture operational patterns. Candidate for ablation due to portability/leakage-proxy risk. |

## Engineered features (created in the notebook)

These features were added to improve signal representation and to capture interpretable “red flags” and interactions.

| Feature | Type | Definition | Intended meaning | Notes |
|---|---|---|---|---|
| `has_past_claims` | binary | PastNumberOfClaims > "none" (or equivalent) | Any claim history | Derived from bins. |
| `strong_process_evidence` | binary | composite flag from process-related fields | Strong procedural inconsistency/evidence | Dataset-specific heuristic. |
| `dow_lag_bin` | categorical/binary | derived time-lag bucket | Timing proxy | Derived. |
| `lag_bin_2` | categorical/binary | chosen lag bucket feature | Delay-related proxy | Selected over lag_bin_3 via baseline CV. |
| `sport_sedan_mismatch` | binary | mismatch flag (domain heuristic) | Potential inconsistency in vehicle category/type | Used in analysis; optional in modeling. |
| `fault_no_police_report` | binary | `Fault=Policy Holder` AND `PoliceReportFiled=No` | Red-flag interaction | Interpretable domain feature. |
| `policy_coverage_combo` | categorical | `PolicyType + "__" + CoverageType` | Product interaction | Helps capture segment-specific risk. |
| `claim_delay_flag` | categorical | bucketed delay proxy from timing fields | Fast/normal/late claim behavior | Simple interpretable bucket. |
| `high_past_claims` | binary | Past claims ≥ 2 (approx.) | “Serial claimer” indicator | Derived from bins / parsing. |
| `high_supplements` | binary | Supplements ≥ 2 (approx.) | High claim updates indicator | Derived from bins / parsing. |
