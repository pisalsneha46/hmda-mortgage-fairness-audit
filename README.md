### Responsible ML Mortgage Approval Classifier Audit

## Project overview

This project was completed for DNSC 6330 — Responsible Machine Learning at George Washington University. The goal was to design, train, and audit a mortgage approval classifier using the 2024 HMDA mortgage application dataset, with a focus on responsible AI principles rather than raw performance alone.

The work centers on building an end‑to‑end, auditable pipeline that can be scrutinized for fairness, robustness, transparency, and governance readiness in a high‑stakes lending context.

---

## Objectives

- **Model development:** Build a supervised learning pipeline for mortgage approval prediction.
- **Fairness evaluation:** Systematically evaluate fairness across protected demographic groups.
- **Failure mode analysis:** Identify and analyze model vulnerabilities and failure modes.
- **Governance assessment:** Assess deployment defensibility under responsible AI principles.
- **Monitoring design:** Recommend monitoring and governance controls for production deployment.

---

## Dataset and target

**Dataset:** 2024 HMDA (Home Mortgage Disclosure Act) public mortgage application data.

**Target variable:**

- **`action_taken`** (binary label constructed for modeling)
  - Approved/Originated → 1
  - Denied → 0

The dataset consists of millions of real mortgage applications. All modeling steps were designed to be reproducible and auditable, with clear separation between raw data ingestion, preprocessing, model training, and evaluation.

### Protected attributes handling

To reduce direct discrimination risk, the following protected attributes were **excluded from model training**:

- **Race**
- **Sex**
- **Ethnicity**
- **Age**

These attributes were retained only for **post‑hoc fairness analysis**, not as model inputs.

---

## Features used

### Numeric features

- **Income**
- **Property Value**
- **Tract‑MSA Percentage**
- **FFIEC MSA Median Income**

### Categorical features

- **Loan Purpose**
- **Loan Type**
- **Occupancy**
- **Lien Status**
- **Submission Factor**
- **Dwelling Factor**
- **State Factor**
- **DTI Bucket**

All features were processed through a structured preprocessing pipeline (encoding, scaling where appropriate, and handling of missing values) to ensure consistent treatment across training and evaluation splits.

---

## Methodology and modeling steps

### 1. Data preparation

- **Data ingestion:** Load 2024 HMDA data and filter to relevant mortgage products and actions.
- **Label construction:** Map HMDA `action_taken` codes into a binary approval/denial target.
- **Train/validation/test split:** Partition data into disjoint sets to support model selection and final evaluation.
- **Feature cleaning:** Handle missing values, outliers, and inconsistent categorical codes.
- **Protected attribute isolation:** Remove protected attributes from the feature matrix while retaining them in a separate structure for fairness analysis.

### 2. Feature engineering

- **Bucketization:** Create **DTI buckets** and other discretized variables where appropriate to stabilize model behavior.
- **Encoding:** Apply categorical encoders (e.g., one‑hot or target encoding, depending on cardinality and model requirements).
- **Numeric transformations:** Standardize or leave numeric features in raw form depending on model type (tree‑based models typically used raw numeric scales).

### 3. Model selection process

Multiple gradient boosting and XGBoost configurations were evaluated using a consistent evaluation framework. For each candidate model, we:

- **Trained** on the same preprocessed training data.
- **Evaluated** on a held‑out validation set using:
  - **AUC**
  - **Log Loss**
  - **False Positive Rate (FPR)**
  - **False Negative Rate (FNR)**
  - **Positive approval rate**
- **Assessed fairness** using subgroup metrics (e.g., Adverse Impact Ratio) on the validation set.
- **Reviewed calibration** to ensure predicted probabilities aligned with observed approval rates.

### 4. Threshold tuning

After selecting a final XGBoost configuration, we:

- **Calibrated the decision threshold** on the validation set.
- **Aligned the model’s approval rate** with the historical approval rate (approximately three‑quarters of applications approved).
- **Documented the chosen threshold** and its rationale for auditability and governance review.

No specific performance or fairness results are reported here; instead, the focus is on the **process** used to reach and justify modeling decisions.

---

## Fairness audit methodology

The fairness audit followed a structured, repeatable procedure:

### 1. Groups evaluated

- **Single‑attribute groups:**
  - Race
  - Ethnicity
  - Sex
  - Age
- **Intersectional groups:**
  - Combinations of race, sex, and age (e.g., race × sex, race × age, sex × age, and three‑way intersections where sample sizes allowed).

### 2. Metrics and rules

- **Primary fairness metric:** Adverse Impact Ratio (AIR).
- **Decision rule:** Four‑Fifths Rule (AIR ≥ 0.80 as a screening threshold).
- **Reference group:** A majority or historically advantaged group was used as the reference for each attribute (e.g., majority race group, younger age band, etc.).

### 3. Procedure

1. **Compute approval rates** for each subgroup using model predictions at the selected threshold.
2. **Calculate AIR** for each subgroup relative to its reference group.
3. **Flag groups** whose AIR falls below the Four‑Fifths threshold.
4. **Extend analysis** to intersectional subgroups to detect compounded disparities.
5. **Document flagged groups** and propose human‑in‑the‑loop or policy interventions for those groups.

This README intentionally omits specific AIR values and group‑level outcomes, focusing instead on the **audit design and workflow**.

---

## Failure mode and robustness analysis

The project systematically explored several potential failure modes and attack surfaces.

### 1. DTI manipulation risk

- **Risk framing:** Debt‑to‑income (DTI) is partially self‑reported and can be manipulated by applicants or intermediaries.
- **Experiment:** Simulate changes in DTI buckets to observe how many denied applications could flip to approved status.
- **Outcome use:** Use these findings to argue for stronger verification of debt obligations and to highlight DTI as a key governance and monitoring feature.

### 2. Label‑flipping poisoning attacks

- **Risk framing:** Training labels can be corrupted (maliciously or accidentally), especially in automated retraining pipelines.
- **Experiment:** Introduce controlled levels of label corruption into the training data and retrain the model.
- **Evaluation:** Compare performance and fairness metrics before and after corruption to identify when fairness violations emerge that may not be visible through standard performance monitoring alone.
- **Governance implication:** Motivate stricter controls on training data access, audit logging, and dual‑control processes for label updates.

### 3. Age and intersectional bias

- **Risk framing:** Even when protected attributes are excluded from training, residual disparities can appear across age and intersectional groups.
- **Procedure:** Use the fairness audit pipeline to identify age‑related and intersectional disparities and to determine which groups should be subject to additional human review or policy safeguards.
- **Mitigation concept:** Propose monitoring and human‑in‑the‑loop review for groups that exhibit persistent disparities.

Again, specific numerical results are intentionally omitted; the emphasis is on **how** these analyses were conducted and how they inform governance.

---

## Deployment and governance framework

The project recommends **conditional deployment** of the model within a robust governance framework, emphasizing continuous oversight rather than one‑time approval.

### Monitoring controls (design)

- **Fairness monitoring:**
  - Periodic AIR monitoring for all protected and intersectional groups.
- **Drift monitoring:**
  - Population Stability Index (PSI) monitoring for key features and score distributions.
- **Performance monitoring:**
  - Tracking AUC and error rates over time to detect degradation.
- **Data and label governance:**
  - Audit logging of training data and label changes.
  - Access controls for any process that can alter training labels or data.

### Pause and escalation triggers (design)

Deployment should be paused and escalated for review if any of the following occur:

- **Fairness:** AIR for any protected group falls below a pre‑defined critical threshold.
- **Performance:** AUC or other key metrics degrade beyond a defined tolerance band.
- **Drift:** PSI exceeds a defined threshold, indicating substantial distribution shift.
- **Security:** A credible data poisoning or integrity incident is identified.

These triggers are meant to be **pre‑committed governance rules** that can be audited and enforced by risk, compliance, or model risk management teams.

## Key Results
- **Model:** XGBoost, n=200 estimators, threshold=0.70
- **Performance:** AUC = 0.8587, Log Loss = 0.3681, zero generalization gap
- **Scale:** 8M+ training records, 837K+ test applicants
- **Fairness:** No single-attribute violations for race/ethnicity/sex
- **Violations found:** Age >74 (AIR=0.789), Black Female (AIR=0.794), NHOPI Female (AIR=0.746)
- **Deployment recommendation:** Conditional — with mandatory human review for flagged subgroups

---

## Repository contents

### Notebooks

- **`RML_Capstone.ipynb`**
  - End‑to‑end model development pipeline: data preparation, feature engineering, model training, threshold tuning, and evaluation.
- **`RML_Capstone_Fairness_Audit.ipynb`**
  - Fairness evaluation workflow: subgroup definition, AIR computation, intersectional analysis, and reporting.

### Presentation

- **`Responsible ML Group 2 Final Presentation.pdf`**
  - High‑level summary of methodology, key design choices, and governance recommendations.

---

## Libraries and tools used

- **Core language and environment**
  - Python
  - Jupyter Notebook

- **Data manipulation**
  - Pandas
  - NumPy

- **Modeling**
  - Scikit‑learn
  - XGBoost

- **Visualization and reporting**
  - Matplotlib
  - (Optionally) Seaborn or other plotting libraries as needed

These libraries were used to implement preprocessing pipelines, model training, evaluation, fairness analysis, and visualization in a reproducible manner.

---

## AI assistance acknowledgement

This project and README benefited from the use of an AI assistant (Claude) for:

- **Drafting and refining documentation language**
- **Structuring sections for clarity and auditability**
- **Suggesting phrasing for responsible AI concepts and governance framing**

All modeling decisions, fairness thresholds, and governance recommendations were determined by the project team. AI assistance was used as a writing and organization aid, not as a decision‑maker for model design or policy choices.

---

## Key responsible AI concepts applied

- **Fairness auditing**
- **Adverse Impact Ratio (AIR)**
- **Four‑Fifths Rule**
- **Responsible deployment**
- **Model governance and monitoring**
- **Human‑in‑the‑loop review**
- **Data poisoning and robustness analysis**
- **Model auditability and documentation**

---

## Conclusion

This project focuses on **how** to build and audit a mortgage approval classifier in a responsible way—emphasizing process, documentation, and governance over specific performance numbers. By separating protected attributes from training, conducting structured fairness audits, probing failure modes, and designing explicit monitoring and pause criteria, the work illustrates what it means to treat a lending model as a governed socio‑technical system rather than a purely technical artifact.
