# AI Solution Design Report
**Domain:** Finance
**Use Case:** Suspicious Transaction Detection

---

## Task 1: Business Domain
**Finance**

---

## Task 2: Business Problem Definition

**Problem:** Banks and financial institutions process millions of transactions daily. A small fraction of these are fraudulent or suspicious — involving money laundering, account takeover, or unauthorized transfers — but identifying them manually is slow and error-prone.

**Users / Stakeholders:**
- Fraud analysts (primary users of the system output)
- Compliance and risk teams
- Bank customers (affected by fraud outcomes)
- Regulators (AML/KYC compliance obligations)

**Current Process:** Analysts apply rule-based systems (e.g., flag transactions above a threshold, in unusual geographies, or outside normal hours) and manually review alerts. Suspicious cases are escalated for investigation.

**Limitations of Current Process:**
- Rule-based systems generate high false-positive rates, overwhelming analysts
- Manual review cannot scale with transaction volume growth
- Static rules fail to catch novel or evolving fraud patterns
- Average resolution time is 21–35 hours (per KPI data), delaying fraud containment
- Error rates between 4–11% indicate inconsistent detection quality

---

## Task 3: AI Task Type

**Task Type:** Anomaly Detection / Classification

**Justification:** Each transaction must be evaluated as suspicious (positive) or legitimate (negative). When labelled historical data is available, this is a binary classification problem. When labels are sparse or unavailable, unsupervised anomaly detection identifies transactions that deviate significantly from a learned normal baseline. Both approaches are complementary and appropriate for this domain.

---

## Task 4: Data Requirement Plan

| Item | Detail |
|------|--------|
| **Type of data** | Structured transaction records |
| **Data format** | Structured (tabular) |
| **Input features** | Transaction amount, timestamp, merchant category, location (country/city), device ID, account age, transaction frequency, time since last transaction, channel (mobile/web/ATM) |
| **Target variable** | `is_suspicious` (binary: 1 = suspicious, 0 = legitimate) |
| **Data collection** | Historical transaction logs from core banking systems; labelled fraud cases from past investigations |
| **Data quality risks** | Class imbalance (fraud is rare, ~0.1–2% of transactions); label noise from unreported fraud; missing values in optional fields (e.g., device ID); data drift as fraud patterns evolve |

---

## Task 5: Model Recommendation

**Recommended Model:** Neural Network (Feed-forward) with an Autoencoder for anomaly pre-screening

**Architecture:**
- **Autoencoder** (unsupervised) — learns a compressed representation of normal transactions; transactions with high reconstruction error are flagged as anomalies
- **Feed-forward Neural Network** (supervised classifier) — trained on labelled fraud/non-fraud data to produce a probability score per transaction

**Why this model:**
- Feed-forward networks handle high-dimensional structured tabular data effectively
- Autoencoders are well-suited to anomaly detection under class imbalance, since they only need to model normal behaviour
- Combining both provides a two-stage pipeline: reduce candidate volume with autoencoder, then score with classifier
- Simpler than sequence models (RNN/LSTM) while still capturing feature interactions missed by rule-based systems

---

## Task 6: Evaluation Plan

**Technical Metrics:**
- Precision — fraction of flagged transactions that are genuinely suspicious
- Recall — fraction of actual suspicious transactions that are caught
- F1-Score — harmonic mean of precision and recall
- AUC-ROC — overall discrimination ability across thresholds
- False Positive Rate — critical for analyst workload management

**Business Metrics (from KPI data):**
- Reduction in manual processing hours (baseline: 330–567 hrs/month)
- Reduction in average resolution time (baseline: 18–45 hours)
- Reduction in error rate (baseline: 4–11%)
- Improvement in customer satisfaction score (baseline: 6.4–7.6)

**Failure Cases:**
- High false negatives: real fraud passes undetected
- High false positives: legitimate users blocked, analyst overload
- Model degradation over time as fraud patterns shift

**Human Review Process:**
- All transactions above a risk score threshold are queued for analyst review
- Analysts validate and provide feedback labels to retrain the model quarterly
- A human decision is required before any account is blocked

---

## Task 7: Responsible AI Considerations

| Risk | Description | Mitigation |
|------|-------------|------------|
| **Bias in data** | Historical fraud labels may reflect biased enforcement (e.g., more scrutiny of certain demographics or geographies) | Audit training data for demographic parity; monitor model outputs by customer segment |
| **Incorrect predictions** | False positives block legitimate customers; false negatives allow fraud to proceed | Maintain a human-in-the-loop review step; never auto-block without analyst confirmation |
| **Privacy concerns** | Transaction data is highly sensitive personal financial information | Data minimisation; role-based access control; encryption at rest and in transit; comply with data protection regulations |
| **Over-reliance on AI** | Analysts may defer entirely to model scores without independent judgement | Train analysts to treat scores as decision support, not decisions; require manual sign-off |
| **Impact on users** | False fraud flags damage customer trust and may prevent legitimate payments | Set up a fast dispute resolution channel; communicate clearly when transactions are held |
| **Model drift** | Fraudsters adapt; model accuracy degrades without retraining | Schedule regular retraining; monitor precision/recall continuously; set alert thresholds for performance drops |

---

## Task 8: Final Solution Summary

| Item | Detail |
|------|--------|
| **Problem** | Manual and rule-based fraud detection in financial transactions is slow, inaccurate, and unable to scale with transaction volume |
| **Proposed AI Solution** | Two-stage pipeline: Autoencoder for anomaly pre-screening + Feed-forward Neural Network classifier to score each transaction for suspicion |
| **Required Data** | Structured transaction history with features: amount, timestamp, location, device, channel, merchant category; labelled fraud/non-fraud cases |
| **Model Recommendation** | Autoencoder (anomaly detection) + Feed-forward Neural Network (binary classification) |
| **Expected Business Impact** | Reduction in manual review hours by ~40–60%; resolution time cut from 35+ hours to under 10 hours; error rate reduced below 3%; improved customer satisfaction |
| **Risks and Mitigation** | Class imbalance → use SMOTE or weighted loss; false positives → human review gate before blocking; privacy → encryption and access control; model drift → quarterly retraining cycle; analyst over-reliance → mandatory human sign-off policy |
