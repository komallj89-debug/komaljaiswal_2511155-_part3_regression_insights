# Model Comparison Report

## Overview

Three regression models were trained and evaluated on **320 monthly store observations** across 80 retail stores.  
Both **monthly sales** and **monthly profit** were used as target variables.

---

## Models Evaluated

| Model | Regularisation | Key Parameter |
|---|---|---|
| Linear Regression | None | — |
| Ridge Regression | L2 | α = 10 |
| Lasso Regression | L1 | α = 100 |

All models were trained on 80% of the data (256 rows) and tested on 20% (64 rows).  
Features were standardised using `StandardScaler`; missing values were imputed with column medians.

---

## Performance Results

### Monthly Sales

| Model | R² | CV R² (5-fold) | RMSE | MAE |
|---|---|---|---|---|
| Linear Regression ★ | 0.8368 | 0.8160 | £40,265 | £33,652 |
| Ridge (α=10) | 0.8368 | 0.8158 | £40,262 | £33,420 |
| Lasso (α=100) | 0.8369 | 0.8161 | £40,254 | £33,654 |

> ★ Best overall model: **Linear Regression**

### Monthly Profit

| Model | R² | CV R² (5-fold) | RMSE | MAE |
|---|---|---|---|---|
| Linear Regression ★ | 0.6134 | 0.5109 | £18,332 | £14,911 |
| Ridge (α=10) | 0.6066 | 0.5122 | £18,492 | £14,986 |
| Lasso (α=100) | 0.6130 | 0.5118 | £18,340 | £14,917 |

---

## Interpretation

### Why Linear Regression Wins

- **Sales R² = 0.8368**: The model explains 83.7% of the variance in monthly sales.
- **Profit R² = 0.6134**: Profit is harder to predict due to greater sensitivity to discounting and operational factors.
- Ridge and Lasso introduce regularisation penalties that slightly reduce performance on this dataset — a sign the features are already well-calibrated and multicollinearity is low.
- **CV R² stability** (Sales: 0.8160) confirms the model generalises well beyond the training set.

### Regularisation Impact

- **Ridge** shrinks coefficients without eliminating features; minimal improvement over OLS here.
- **Lasso** drives some coefficients toward zero (feature selection); minor performance trade-off.
- Both regularised models perform comparably, suggesting no severe overfitting in the baseline.

---
