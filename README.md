# Credit Approval & Risk Analytics Project

## Overview

This project analyzes a synthetic but economically realistic **loan approval dataset** to study how credit decisions are made, how risk concentrates within a portfolio, and how data-driven models can improve approval, pricing, and risk governance.

The objective is **not** to estimate regulatory credit risk parameters, but to:

* Understand approval behavior
* Identify concentration and tail risk
* Benchmark and improve decision models
* Translate model outputs into **actionable credit decisions** (approve, decline, price, review)

The project follows a **full credit decisioning pipeline**:
**Policy rules → Portfolio diagnostics → Statistical modeling → Calibration → Threshold optimization → Risk‑based pricing**.

---

## Dataset Description

### Structure

* **Observation unit**: Individual loan application
* **Target variable**: Loan approval outcome (Approved / Rejected)
* **Scope**: Retail-style unsecured and semi‑secured lending

### Key Feature Categories

* **Borrower quality**: credit score, past delinquencies
* **Affordability**: income, debt‑to‑income ratio, payment‑to‑income ratios
* **Loan characteristics**: loan amount, purpose, term
* **Stability indicators**: employment length, housing status

The dataset is designed to reflect **real underwriting logic**:

* Credit score and affordability are dominant drivers
* Extreme risk profiles are mostly filtered by policy rules
* Approval decisions are non‑linear and exposure‑sensitive

---

## Credit Approval: Conceptual Background

Loan approval is fundamentally a **risk‑reward trade‑off**:

* Approving more loans increases revenue
* Approving the wrong loans concentrates risk and losses

In practice, approval decisions are not driven by a single model, but by a **layered system**:

1. **Hard policy rules**
   Exclude structurally risky applications (e.g. very low credit score combined with high leverage).

2. **Risk ranking models**
   Rank remaining applicants by relative credit risk.

3. **Decision thresholds**
   Convert risk scores into approve / reject decisions.

4. **Risk‑based pricing**
   Adjust interest rates to compensate for residual risk.

This project mirrors that structure using the information available in the dataset.

---

## Project Structure & Methodology

### 1. Portfolio Diagnostics & Concentration Analysis

Before modeling, the portfolio is analyzed to understand **where exposure is concentrated**.

Key findings:

* Medium and Low credit score buckets account for ~65% of loans and ~63% of total exposure.
* Good and Excellent borrowers represent fewer loans but higher average exposure per loan.
* Very Low credit score borrowers form a small share of the portfolio, limiting extreme tail risk.
* Loan exposure is **highly concentrated in very large loan amounts**, creating a clear concentration risk driven by a small number of high‑value loans.

This step establishes that portfolio risk is primarily **concentration‑driven**, not volume‑driven.

---

### 2. Hard Risk Rules (Policy Layer)

Hard risk rules identify applicants with structurally weak profiles (e.g. low credit score combined with high debt‑to‑income).

Results:

* ~12% of applications fall under hard rules.
* 80% of these applicants are rejected, indicating strong alignment between policy and observed decisions.
* 20% are still approved, revealing measurable policy leakage.
* Hard‑rule applications involve larger loan amounts on average.
* Full enforcement of the rules would remove ~17% of total portfolio exposure.

**Insight:** rule‑based screening disproportionately removes exposure and materially reduces concentration and tail risk before any model‑based decisioning.

---

### 3. Risk Definition & Segmentation

Risk is defined as the **inverse of predicted approval probability**:

* High approval probability → low risk
* Low approval probability → high risk

Applicants are segmented into risk buckets to support:

* Monitoring
* Threshold analysis
* Pricing decisions

Results:

* High‑risk segment represents ~5% of total exposure.
* Majority of the portfolio lies in a medium‑risk band.
* Conservative segmentation compresses strong borrowers, motivating further model refinement.

---

### 4. Baseline Logistic Regression (Eligible Population)

A logistic regression model is trained **only on policy‑eligible applications**, ensuring the model captures **residual risk after governance rules**.

Results:

* AUC = **0.914**, indicating strong discrimination.
* Under a model‑only decision rule, ~9.9% of total exposure would be allocated to risky approvals.
* Policy flags are not statistically significant once continuous variables are included, but remain essential for governance and monitoring.

This model provides a transparent and interpretable baseline for credit decisioning.

---

### 5. Non‑Linear Feature Engineering

To capture non‑linear risk effects while preserving interpretability:

* Logarithmic and polynomial transformations are applied to key drivers.

Results:

* AUC improves from **0.9141 → 0.9327** (+2.02% relative improvement).
* Transformed features exhibit lower skewness and improved stability.
* ROC gains are strongest in the 5–20% false‑positive region, where operational thresholds are typically set.

This demonstrates that meaningful risk structure is non‑linear but monotonic.

---

### 6. Random Forest Benchmark

A Random Forest model is trained to test whether additional non‑linear structure remains.

Results:

* AUC = **0.9604**, the strongest discriminatory performance.
* Feature importance confirms economically intuitive drivers:

  * Credit score (35%)
  * Debt‑to‑income ratio (26%)
  * Recent delinquencies (8%)

Loan size is less predictive than borrower behavior, reinforcing underwriting intuition.

---

### 7. Decision Comparison at Fixed Threshold

At a 0.5 decision threshold:

* Random Forest reduces risky approvals to **$28.6M** of exposure

  * vs $41.1M (baseline logistic regression)
  * vs $32.5M (transformed logistic regression)
* Missed good borrowers drop to **542 loans**, with ~$28.6M in opportunity cost.
* Approval rate remains stable at ~60%.

This represents a **Pareto improvement**: lower risk and better opportunity capture without tightening credit.

---

### 8. Probability Calibration

Calibration diagnostics reveal significant miscalibration in raw model probabilities.

* Pre‑calibration average error: ~6–6.8%
* Isotonic recalibration applied
* Post‑calibration error reduced to ~0.5%

Calibrated probabilities align predicted and observed approval rates across risk buckets, enabling reliable thresholding and pricing.

---

### 9. Threshold Optimization

Using calibrated probabilities, the approval threshold is optimized.

* Threshold adjusted from 0.50 to **0.88**
* Results:

  * ~27% more marginal rejections
  * ~80% reduction in bad approvals
  * ~$18.8M reduction in risky exposure
  * Decision quality improves from 90.5% to 97.4%

This highlights threshold selection as a **high‑impact, low‑cost risk lever**.

---

### 10. Risk‑Based Pricing

Model outputs are translated into pricing decisions.

Key outcomes:

* Clear monotonic relationship between approval probability and suggested interest rates.
* Low‑risk borrowers are overcharged under current pricing and could be repriced downward.
* Medium‑risk segments show smooth, coherent differentiation.
* High‑risk applicants are effectively priced out, confirming pricing as a secondary risk control.

Pricing decisions naturally separate applications into:

* Decline
* Price
* Price with tighter terms
* Review

---

### 11. Production‑Ready Decision Logic

The final output is an API‑compatible scoring framework:

**Applicant → Approval probability → Risk tier → Decision → Pricing action**

Example:

> Approval probability ≈ 66% → Risk tier D → Approve only with tighter pricing or additional review

---

## Conclusion

This project demonstrates a complete, realistic credit decisioning workflow:

* Portfolio risk is driven by concentration and large exposures
* Hard rules remove disproportionate tail risk
* Statistical models rank residual risk effectively
* Non‑linear structure materially improves decisions
* Calibration enables reliable thresholds and pricing
* Threshold and pricing choices drive economic outcomes

The result is a coherent framework for **risk‑aware, exposure‑sensitive loan approval and pricing**.

