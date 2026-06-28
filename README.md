# Part 3: Regression-Based Business Insights

A complete regression analysis pipeline on retail store performance data,
delivering actionable business intelligence through multiple regression models
and data visualisations.

---

## Project Overview

This project analyses **320 monthly observations** across **80 retail stores**
(4 store types × 4 regions × 4 months) to:

1. Build and compare regression models that predict **monthly sales** and **monthly profit**
2. Identify the most influential operational drivers
3. Surface strategic insights by store type and region

---

## Dataset — `business_regression_data.xlsx`

| Column | Description |
|---|---|
| `store_id` | Unique store identifier |
| `month` | Observation month |
| `region` | East / North / South / West |
| `store_type` | Airport / High Street / Mall / Residential |
| `marketing_spend` | Monthly marketing budget (£) |
| `footfall` | Monthly customer visits |
| `avg_discount_pct` | Average discount applied |
| `staff_count` | Number of staff on shift |
| `inventory_availability_pct` | % of SKUs in stock |
| `competitor_distance_km` | Distance to nearest competitor (km) |
| `holiday_flag` | 1 = holiday month, 0 = regular |
| `customer_rating` | Average customer rating (1–5) |
| `monthly_sales` | **Target** – total monthly revenue (£) |
| `monthly_profit` | **Target** – total monthly profit (£) |

---

## Models Compared

| Model | Notes |
|---|---|
| Linear Regression | Baseline OLS |
| Ridge (α=10) | L2 regularisation |
| Lasso (α=100) | L1 regularisation / feature selection |

---

## Key Findings

### Model Performance (Linear Regression – best overall)
| Target | R² | CV R² (5-fold) | RMSE |
|---|---|---|---|
| Monthly Sales | **0.84** | 0.82 | £40,265 |
| Monthly Profit | **0.61** | 0.51 | £18,332 |

### Top Sales Drivers
1. **Footfall** – by far the strongest predictor of revenue
2. **Marketing Spend** – meaningful ROI signal
3. **Inventory Availability %** – stock-outs directly suppress sales

### Top Profit Drivers
1. **Footfall** – high traffic lifts profit proportionally
2. **Avg Discount %** – discounting erodes margin significantly
3. **Inventory Availability %** – full shelves retain high-margin customers

### Business Insights
- **Airport stores** generate the highest average monthly profit margin (~16.8%)
- **South region** leads in both sales (£693k avg) and profit (£115k avg)
- Customer rating shows a positive correlation with profit margin
- Competitor proximity has limited impact at the portfolio level

---

## Outputs

| File | Description |
|---|---|
| `fig1_model_comparison.png` | R², CV R², and RMSE across all three models |
| `fig2_feature_importance.png` | Permutation importance for Sales and Profit |
| `fig3_actual_vs_predicted.png` | Scatter plots of actual vs. predicted values |
| `fig4_business_insights.png` | Six-panel business intelligence dashboard |
| `fig5_correlation_heatmap.png` | Full feature correlation matrix |

---

## Setup & Usage

```bash
# 1. Clone the repo
git clone <YOUR_GITHUB_REPO_URL>
cd <repo-name>

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run the analysis
python regression_analysis.py
```

All five output figures are generated in the working directory.

---

## Requirements

See `requirements.txt`. Python 3.9+ recommended.

---


