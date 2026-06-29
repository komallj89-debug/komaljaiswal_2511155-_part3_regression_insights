# Model Equations

## Methodology

All features were standardised (mean=0, std=1) before fitting.  
Missing values were imputed using column medians.  
Categorical variables (region, store_type) were label-encoded.

---

## Simple Linear Regression

### Monthly Sales ~ Footfall (n=320)

| Parameter | Value |
|---|---|
| Intercept (β₀) | £446,410.58 |
| Footfall coefficient (β₁) | £35.6780 per visitor |
| R² | 0.7363 |

**Interpretation:** Each additional monthly visitor generates approximately  
**£35.68** in additional monthly sales.

---

## Multiple Linear Regression — Monthly Sales

**Equation (standardised features):**

```
Ŷ_sales = β₀ + β₁·Marketing_Spend + β₂·Footfall + β₃·Avg_Discount
         + β₄·Staff_Count + β₅·Inventory_Avail + β₆·Competitor_Dist
         + β₇·Holiday_Flag + β₈·Customer_Rating + β₉·Region + β₁₀·Store_Type
```

### Coefficients (standardised)

| Feature | Coefficient | Direction |
|---|---|---|
| Marketing Spend | +24164.37 | ↑ Positive |
| Footfall | +68888.59 | ↑ Positive |
| Avg Discount % | -3492.08 | ↓ Negative |
| Staff Count | +17009.73 | ↑ Positive |
| Inventory Avail % | +15792.78 | ↑ Positive |
| Competitor Distance km | -13922.48 | ↓ Negative |
| Holiday Flag | +5733.42 | ↑ Positive |
| Customer Rating | +6757.03 | ↑ Positive |
| Region | +9977.37 | ↑ Positive |
| Store Type | -11137.23 | ↓ Negative |
| Intercept | 680,628.72 | — |

**Model R² = 0.8479**

---

## Multiple Linear Regression — Monthly Profit

### Coefficients (standardised)

| Feature | Coefficient | Direction |
|---|---|---|
| Marketing Spend | +3689.19 | ↑ Positive |
| Footfall | +14493.36 | ↑ Positive |
| Avg Discount % | -7830.28 | ↓ Negative |
| Staff Count | +2107.56 | ↑ Positive |
| Inventory Avail % | +7615.07 | ↑ Positive |
| Competitor Distance km | -2852.17 | ↓ Negative |
| Holiday Flag | -2004.07 | ↓ Negative |
| Customer Rating | +175.52 | ↑ Positive |
| Region | +1895.68 | ↑ Positive |
| Store Type | -2950.95 | ↓ Negative |
| Intercept | 111,568.20 | — |

**Model R² = 0.5712**

---

## Ridge Regression (α=10)

Ridge adds an L2 penalty: minimises `RSS + α·Σβ²`  
Coefficients are shrunk toward zero proportionally — reduces variance with slight bias increase.  
Useful when multicollinearity is suspected between features.

## Lasso Regression (α=100)

Lasso adds an L1 penalty: minimises `RSS + α·Σ|β|`  
Some coefficients are driven exactly to zero, performing implicit feature selection.  
At α=100, lower-importance features (competitor distance, holiday flag) may be zeroed.

---

## Notes on Standardisation

To interpret a standardised coefficient in original units:

```
real_coefficient = standardised_coefficient × std(target) / std(feature)
```

Example — Footfall impact on Sales:
- Standardised β = +68888.5901
- Footfall std = 2,496 visitors/month
- Sales std = £103,772
- Real impact ≈ **£2,864,343.50 per additional visitor/month**
