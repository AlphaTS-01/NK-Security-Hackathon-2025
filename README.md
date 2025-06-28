# NK Securities Hackathon 2025 — Solution Documentation

**Author:** Tejas Shrivastava  
**Date:** June 2025

---

## 📌 Overview

The NK Securities Hackathon challenge focused on predicting **implied volatilities (IV)** for different strike prices where the **training and test sets had mismatched strike columns** — demanding robust interpolation and extrapolation of the volatility surface.

This repo documents my complete solution journey, which evolved through three conceptual phases:  
1️⃣ Parametric Curve Fitting  
2️⃣ Autoencoder-Based Imputation  
3️⃣ MICE-Based Iterative Imputation (Final Solution)

---

## ⚙️ Repo Structure

```
NK-Security-Hackathon-2025/
│
├── Datasets/           # Raw & processed data files
├── Experimentations/   # Jupyter notebooks for each phase
├── Final Submission/   # MICE
├── documentation.pdf   # compiled methods and results
└── README.md           # 📄 This file

```

---

## 🧩 Problem Understanding

> **Goal:**  
> Predict missing strike-level IVs for options with partially available data and weak market features.
> Mismatched strikes required more than standard supervised learning.

---

## 🔬 Phase 1 — Parametric Curve Fitting

**Methodology:**  
- Fit well-known curves (Quadratic, SVI) for each training IV vector.
- Interpolate IVs for test strikes.
- Train boosting models (XGBoost, LightGBM) using market features.

**Result:**  
Low performance (`Public Score ~ 10⁻³`).
- Did not leverage known IVs in the test data.
- Market features were not predictive enough.

---

## 🤖 Phase 2 — Autoencoder-Based Imputation

**Methodology:**  
- Randomly mask parts of IV vectors during training.
- Train a denoising autoencoder to learn reconstruction.
- Impute missing test IVs with the trained model.

**Key Improvements:**  
- Developed **edge-focused masking** to mimic real strike missingness.
- Mutual Information and Pearson correlation proved market features were weak (MI ≈ 0.12).

**Result:**  
Better than Phase 1 (`Public Score ~ 10⁻⁵ to 10⁻⁶`) but not final.

---

## 🔁 Phase 3 — MICE-Based Iterative Imputation (Final Solution)

> **Breakthrough:**  
> Treat missing IVs as supervised targets and iteratively predict them using all available IVs.

### 📚 What is MICE?

**Multiple Imputation by Chained Equations (MICE):**  
1. Fill missing values with initial guesses (e.g., median).  
2. For each column with missing data:
   - Use other columns as predictors.
   - Train a regressor on rows with values present.
   - Predict the missing values.
3. Repeat until convergence.

---

### ✅ Why MICE Was Optimal

- Uses all test IVs together.
- Captures strike-level dependencies.
- Adapts to irregular missing patterns.
- No restrictive parametric assumptions.

---

### ⚙️ Final Architecture

- Applied **MICE** only on test data (training data excluded).
- Dropped market features completely.
- Used **ExtraTreesRegressor** for stable iterative updates.
- Tuned for 5–10 iterations.
- Final ensemble averaged multiple MICE runs.

**Result:**  
Best `Public Score ~ 10⁻⁷`.

---

## 🏆 Key Insights

- **Phase 1:** Confirmed need to leverage test info fully.
- **Phase 2:** Proved that market features had limited signal.
- **Phase 3:** MICE + trees matched the problem’s structure best.

---
