# drift-monitoring

### Model Drift & Data Drift: What to Monitor and When to Retrain Machine Learning Model in Production

A practical guide and reference implementation for:
- Detecting **data drift**, **model drift**, and **concept drift**
- Understanding why **data drift alone is not enough** to trigger retraining
- Designing a **retrain decision policy** that balances model quality, cost, and risk

This repository focuses on **monitoring correctness**, not just metric calculation.

---

### Why this repository exists

Many production ML systems rely on a simple rule:

> “If data drift exceeds a threshold → retrain the model”

This approach is incomplete and often harmful.

- Data drift does **not always degrade model performance**
- Model performance can degrade **without visible data drift**
- Retraining is expensive and introduces operational risk
- Retraining on unstable or broken data can make models worse

This repository demonstrates:
- What different types of drift actually mean
- What each drift signal can and cannot tell you
- How to combine drift, performance, and data quality signals into a **sound retraining decision**

---

### Core concepts

### 1. Data Drift (Covariate Shift)

**Definition:**  
The statistical distribution of input features changes over time.

**Examples:**
- Change in user demographics
- Shift in traffic sources
- New product flows or feature values
- Seasonality effects

**Common detection methods:**
- Population Stability Index (PSI)
- KS test (numerical features)
- Chi-square test (categorical features)
- KL / JS divergence
- Embedding distribution drift (text, images)

**What data drift tells you:**
- Something upstream has changed
- Feature distributions no longer match training data

**What data drift does NOT tell you:**
- Whether the model is performing worse
- Whether retraining will improve outcomes

---

### 2. Model Drift (Performance Drift)

**Definition:**  
The model’s predictive quality degrades in production.

**Examples:**
- Precision/recall drops for fraud detection
- Calibration worsens
- Regression error increases
- Business KPIs decline

**Detection signals:**
- Performance metrics once labels arrive (AUC, F1, RMSE, calibration error)
- Prediction confidence shifts (overconfidence / underconfidence)
- Proxy metrics when labels are delayed (conversion rate, complaints, chargebacks)

**This is the primary reason to retrain models.**

---

### 3. Concept Drift

**Definition:**  
The relationship between inputs (X) and the target (y) changes.

**Examples:**
- User behavior changes due to pricing or policy updates
- Fraud patterns evolve while feature distributions stay similar
- Label definition changes over time

**Key property:**
- Concept drift may occur **with or without data drift**
- It is often invisible to pure distribution-based metrics

---

## Why data drift alone is not enough to trigger retraining

### Scenario 1: Data drift with no performance impact

- Drift occurs in low-importance features
- Core predictive signals remain stable
- Model performance stays consistent

**Result:**  
Retraining adds cost without benefit and may introduce regressions.

---

### Scenario 2: Performance degradation without obvious data drift

- Concept drift alters decision boundaries
- Labels change meaning
- Instrumentation or feedback loops affect outcomes

**Result:**  
Data drift metrics remain calm while real-world performance degrades.

---

### Scenario 3: Data quality issues misinterpreted as drift

- Missing values spike
- Feature leakage disappears
- Schema changes propagate silently

**Result:**  
Retraining on corrupted data accelerates failure.

---

### Recommended retraining framework

Retraining should be a **decision**, not an automatic reaction.

### Layer 1: Data integrity checks (blocking layer)

Do **not** retrain if:
- Schema mismatch exists
- Missing values spike abnormally
- Training-serving skew is detected
- Feature computation changes unexpectedly

**Action:**  
Fix data pipelines first.

---

### Layer 2: Drift signals (monitoring layer)

Use drift metrics to:
- Detect upstream changes
- Identify feature-level instability
- Prioritize monitoring for important features

**Action:**  
Increase observation, prepare candidate models, but do not retrain automatically.

---

### Layer 3: Performance and impact signals (decision layer)

Retrain when:
- Performance metrics degrade beyond tolerance (once labels arrive)
- Proxy metrics consistently decline
- Backtesting shows improvement over current production model

**Action:**  
Retrain with validation, safe rollout, and rollback capability.

---

### Drift metrics
- PSI (numeric and categorical)
- KS test
- Chi-square test
- JS divergence
- Prediction distribution drift

### Simulated drift scenarios
- Pure data drift (distribution shift only)
- Concept drift (target relationship changes)
- Delayed label performance drift

### Retrain decision logic
A reference policy that combines:
- Drift severity
- Feature importance
- Prediction behavior
- Data quality signals
- Delayed ground truth

---

