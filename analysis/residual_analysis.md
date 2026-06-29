# Residual Analysis

## Purpose

Residual analysis validates that the regression model satisfies core OLS assumptions:  
1. **Linearity** — residuals scatter randomly around zero  
2. **Homoscedasticity** — constant variance across fitted values  
3. **Normality** — residuals approximately normally distributed  
4. **Independence** — no systematic patterns in errors

---

## Model: Linear Regression

### Monthly Sales Residuals

| Metric | Value |
|---|---|
| Mean Residual | £-3,864.69 |
| Std Deviation | £40,079.23 |
| Min Residual | £-100,242.00 |
| Max Residual | £69,985.34 |
| % Within ±£50k | 79.7% |

### Monthly Profit Residuals

| Metric | Value |
|---|---|
| Mean Residual | £831.69 |
| Std Deviation | £18,313.00 |
| Min Residual | £-39,801.98 |
| Max Residual | £44,806.82 |
| % Within ±£20k | 71.9% |

---

## Diagnostic Assessment

### 1. Linearity
Residual-vs-fitted plots (see `screenshots/residuals_preview.png`) show random scatter with no clear curvature,  
supporting the linearity assumption for monthly sales.  
Profit shows slightly more spread at higher fitted values — mild heteroscedasticity may exist.

### 2. Homoscedasticity
Sales residuals maintain relatively constant variance across the fitted range (£400k–£950k).  
Profit residuals show modest fanning, suggesting a log-transform of the target could improve fit.

### 3. Normality
Both residual histograms approach bell-curve shape with near-zero means:
- Sales mean residual: £-3,865 (effectively zero)
- Profit mean residual: £832 (effectively zero)

A formal Shapiro-Wilk or Kolmogorov-Smirnov test would confirm normality at α=0.05.

### 4. Independence
No temporal or store-level clustering pattern is visible in residuals; the dataset's cross-sectional  
structure reduces autocorrelation risk.

---

## Outlier Assessment

Residuals beyond ±2σ may indicate unusual store months:

- **Sales**: σ = £40,079 → flag residuals outside ±£80,158
- **Profit**: σ = £18,313 → flag residuals outside ±£36,626

These should be reviewed for data entry errors, exceptional trading events, or structural breaks.

---

## Conclusion

The residual diagnostics are broadly satisfactory for the sales model.  
The profit model would benefit from additional features (e.g., cost structure, promotional spend ROI)  
or a non-linear modelling approach to reduce unexplained variance.
