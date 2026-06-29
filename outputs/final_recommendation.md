# Final Business Recommendation

## Executive Summary

Regression analysis of 320 monthly observations across 80 retail stores reveals that **footfall is the  
dominant driver of both monthly sales and profit**. The best-performing model (Linear Regression) achieves an  
R² of **0.84 for sales** and **0.61 for profit**, enabling reliable revenue forecasting  
and targeted operational improvements.

---

## Key Findings

### Top Drivers of Monthly Sales

| Rank | Feature | Permutation Importance |
|---|---|---|
| 1 | Footfall | 0.7713 |
| 2 | Marketing Spend | 0.1252 |
| 3 | Inventory Avail % | 0.0634 |

### Top Drivers of Monthly Profit

| Rank | Feature | Permutation Importance |
|---|---|---|
| 1 | Footfall | 0.3976 |
| 2 | Avg Discount % | 0.1420 |
| 3 | Inventory Avail % | 0.1181 |

---

## Store Type Analysis

| Store Type | Avg Monthly Sales | Avg Monthly Profit | Profit Margin |
|---|---|---|---|
| Airport | £715,944 | £120,392 | 16.8% |
| Mall | £697,425 | £112,486 | 16.1% |
| Residential | £673,773 | £109,391 | 16.2% |
| High Street | £667,010 | £110,713 | 16.5% |

---

## Regional Analysis

| Region | Avg Monthly Sales | Avg Monthly Profit |
|---|---|---|
| South | £693,320 | £114,586 |
| West | £680,681 | £113,018 |
| East | £680,382 | £109,746 |
| North | £669,056 | £109,616 |

---

## Strategic Recommendations

### 1. Prioritise Footfall Growth
Footfall is the strongest predictor of both sales and profit. Invest in:
- In-store events, loyalty programmes, and local marketing to drive traffic
- Location scouting for new stores in high-footfall catchments

### 2. Optimise Discounting Strategy
Average discount % is the **second-biggest drag on profit margin**.  
- Introduce targeted promotions rather than blanket discounting
- Set discount guardrails per store type (Airport stores should discount minimally)

### 3. Maintain Inventory Availability Above 90%
Inventory availability consistently appears in the top 3 drivers for both targets.  
- Target ≥ 90% SKU availability across all store types
- Improve demand forecasting to reduce stock-outs without excess holding costs

### 4. Expand Airport Format
Airport stores achieve the highest profit margin (16.8%).  
Explore opportunities to grow the Airport store count where viable.

### 5. Investigate South Region Outperformance
South region leads in both sales and profit. Conduct a root-cause analysis to  
identify transferable best practices for East and North regions.

---

## Model Confidence

| Target | R² | CV R² | RMSE |
|---|---|---|---|
| Monthly Sales | 0.8368 | 0.8160 | £40,265 |
| Monthly Profit | 0.6134 | 0.5109 | £18,332 |

The sales model is production-ready. The profit model provides directional guidance  
but should be supplemented with cost-side data for higher precision.
