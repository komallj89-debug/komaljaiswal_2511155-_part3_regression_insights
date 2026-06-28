"""
Part 3: Regression-Based Business Insights
==========================================
Analyzes store performance data using multiple regression techniques
to derive actionable business insights.
"""

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import seaborn as sns
import warnings
warnings.filterwarnings("ignore")

from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.impute import SimpleImputer
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import r2_score, mean_squared_error, mean_absolute_error
from sklearn.inspection import permutation_importance

# ── 0. Style ─────────────────────────────────────────────────────────────────
plt.rcParams.update({
    "figure.facecolor": "#0f1117",
    "axes.facecolor":   "#1a1d27",
    "axes.edgecolor":   "#3a3d4e",
    "axes.labelcolor":  "#c9d1d9",
    "axes.titlecolor":  "#e6edf3",
    "xtick.color":      "#8b949e",
    "ytick.color":      "#8b949e",
    "text.color":       "#c9d1d9",
    "grid.color":       "#21262d",
    "grid.linestyle":   "--",
    "grid.alpha":       0.5,
    "legend.facecolor": "#161b22",
    "legend.edgecolor": "#3a3d4e",
    "font.family":      "DejaVu Sans",
})

ACCENT   = "#58a6ff"
GREEN    = "#3fb950"
ORANGE   = "#f0883e"
RED      = "#f85149"
PURPLE   = "#bc8cff"
YELLOW   = "#e3b341"
PALETTE  = [ACCENT, GREEN, ORANGE, PURPLE, YELLOW, RED]

# ── 1. Load & Clean ───────────────────────────────────────────────────────────
print("Loading data …")
df = pd.read_excel("business_regression_data.xlsx")

# Fill missing values with column medians (numeric only)
for col in df.select_dtypes(include=[np.number]).columns:
    df[col].fillna(df[col].median(), inplace=True)

# Encode categoricals
le_region = LabelEncoder()
le_type   = LabelEncoder()
df["region_enc"]     = le_region.fit_transform(df["region"])
df["store_type_enc"] = le_type.fit_transform(df["store_type"])

# Feature set
FEATURES = [
    "marketing_spend", "footfall", "avg_discount_pct", "staff_count",
    "inventory_availability_pct", "competitor_distance_km", "holiday_flag",
    "customer_rating", "region_enc", "store_type_enc",
]
TARGET_SALES  = "monthly_sales"
TARGET_PROFIT = "monthly_profit"

X  = df[FEATURES]
ys = df[TARGET_SALES]
yp = df[TARGET_PROFIT]

imputer = SimpleImputer(strategy="median")
X_imp = imputer.fit_transform(X)

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_imp)

X_train_s, X_test_s, ys_train, ys_test = train_test_split(X_scaled, ys, test_size=0.2, random_state=42)
X_train_p, X_test_p, yp_train, yp_test = train_test_split(X_scaled, yp, test_size=0.2, random_state=42)


# ── 2. Train Models ───────────────────────────────────────────────────────────
print("Training models …")

models = {
    "Linear Regression": LinearRegression(),
    "Ridge (α=10)":      Ridge(alpha=10),
    "Lasso (α=100)":     Lasso(alpha=100, max_iter=5000),
}

results = {}
for name, mdl in models.items():
    # Sales
    mdl.fit(X_train_s, ys_train)
    s_pred  = mdl.predict(X_test_s)
    s_r2    = r2_score(ys_test, s_pred)
    s_rmse  = np.sqrt(mean_squared_error(ys_test, s_pred))
    s_mae   = mean_absolute_error(ys_test, s_pred)
    s_cv    = cross_val_score(mdl, X_scaled, ys, cv=5, scoring="r2").mean()

    # Profit (refit same type of model)
    mdl2 = type(mdl)(**mdl.get_params())
    mdl2.fit(X_train_p, yp_train)
    p_pred  = mdl2.predict(X_test_p)
    p_r2    = r2_score(yp_test, p_pred)
    p_rmse  = np.sqrt(mean_squared_error(yp_test, p_pred))
    p_mae   = mean_absolute_error(yp_test, p_pred)
    p_cv    = cross_val_score(mdl2, X_scaled, yp, cv=5, scoring="r2").mean()

    results[name] = {
        "sales_model":  mdl,
        "profit_model": mdl2,
        "sales":  {"r2": s_r2, "rmse": s_rmse, "mae": s_mae, "cv_r2": s_cv, "pred": s_pred},
        "profit": {"r2": p_r2, "rmse": p_rmse, "mae": p_mae, "cv_r2": p_cv, "pred": p_pred},
    }

# Best model = highest mean R² across both targets
best_name = max(results, key=lambda n: (results[n]["sales"]["r2"] + results[n]["profit"]["r2"]) / 2)
best      = results[best_name]

print(f"\nBest model: {best_name}")
print(f"  Sales  R²={best['sales']['r2']:.4f}  CV R²={best['sales']['cv_r2']:.4f}  RMSE={best['sales']['rmse']:,.0f}")
print(f"  Profit R²={best['profit']['r2']:.4f}  CV R²={best['profit']['cv_r2']:.4f}  RMSE={best['profit']['rmse']:,.0f}")


# ── 3. Feature importance (permutation) ──────────────────────────────────────
perm_s = permutation_importance(best["sales_model"],  X_test_s, ys_test, n_repeats=20, random_state=42)
perm_p = permutation_importance(best["profit_model"], X_test_p, yp_test, n_repeats=20, random_state=42)

fi_sales  = pd.Series(perm_s.importances_mean, index=FEATURES).sort_values(ascending=False)
fi_profit = pd.Series(perm_p.importances_mean, index=FEATURES).sort_values(ascending=False)

FEATURE_LABELS = {
    "marketing_spend":            "Marketing Spend",
    "footfall":                   "Footfall",
    "avg_discount_pct":           "Avg Discount %",
    "staff_count":                "Staff Count",
    "inventory_availability_pct": "Inventory Availability %",
    "competitor_distance_km":     "Competitor Distance (km)",
    "holiday_flag":               "Holiday Flag",
    "customer_rating":            "Customer Rating",
    "region_enc":                 "Region",
    "store_type_enc":             "Store Type",
}


# ── 4. Business Aggregates ────────────────────────────────────────────────────
by_type   = df.groupby("store_type")[["monthly_sales", "monthly_profit"]].mean()
by_region = df.groupby("region")[["monthly_sales", "monthly_profit"]].mean()

profit_margin = df.copy()
profit_margin["profit_margin_pct"] = (df["monthly_profit"] / df["monthly_sales"]) * 100
margin_by_type   = profit_margin.groupby("store_type")["profit_margin_pct"].mean()
margin_by_region = profit_margin.groupby("region")["profit_margin_pct"].mean()


# ── 5. FIGURE 1 – Model Comparison ───────────────────────────────────────────
print("\nGenerating Figure 1: Model Comparison …")
fig, axes = plt.subplots(1, 3, figsize=(16, 6))
fig.patch.set_facecolor("#0f1117")
fig.suptitle("Model Performance Comparison", fontsize=16, fontweight="bold", color="#e6edf3", y=1.02)

metric_labels = ["R²", "CV R²", "RMSE (÷1000)"]
model_names   = list(results.keys())
colors        = [ACCENT, GREEN, ORANGE]

for ax_idx, (target, t_label) in enumerate([("sales", "Monthly Sales"), ("profit", "Monthly Profit")]):
    pass  # We'll do a combined bar chart

# Bar chart: R² for all models, both targets
ax = axes[0]
x   = np.arange(len(model_names))
w   = 0.35
r2s = [results[n]["sales"]["r2"]  for n in model_names]
r2p = [results[n]["profit"]["r2"] for n in model_names]
ax.bar(x - w/2, r2s, w, label="Sales",  color=ACCENT,  alpha=0.85)
ax.bar(x + w/2, r2p, w, label="Profit", color=GREEN,   alpha=0.85)
ax.set_xticks(x); ax.set_xticklabels([n.split("(")[0].strip() for n in model_names], fontsize=9)
ax.set_ylabel("R² Score"); ax.set_title("R² Score"); ax.legend()
ax.set_ylim(0, 1.05); ax.axhline(0.7, color="#8b949e", linestyle=":", alpha=0.7, label="0.7 threshold")

# CV R²
ax = axes[1]
cv_s = [results[n]["sales"]["cv_r2"]  for n in model_names]
cv_p = [results[n]["profit"]["cv_r2"] for n in model_names]
ax.bar(x - w/2, cv_s, w, label="Sales",  color=ACCENT, alpha=0.85)
ax.bar(x + w/2, cv_p, w, label="Profit", color=GREEN,  alpha=0.85)
ax.set_xticks(x); ax.set_xticklabels([n.split("(")[0].strip() for n in model_names], fontsize=9)
ax.set_ylabel("Cross-Val R²"); ax.set_title("5-Fold Cross-Validation R²"); ax.legend()
ax.set_ylim(0, 1.05)

# RMSE
ax = axes[2]
rm_s = [results[n]["sales"]["rmse"]  / 1000 for n in model_names]
rm_p = [results[n]["profit"]["rmse"] / 1000 for n in model_names]
ax.bar(x - w/2, rm_s, w, label="Sales",  color=ACCENT, alpha=0.85)
ax.bar(x + w/2, rm_p, w, label="Profit", color=GREEN,  alpha=0.85)
ax.set_xticks(x); ax.set_xticklabels([n.split("(")[0].strip() for n in model_names], fontsize=9)
ax.set_ylabel("RMSE (÷1 000)"); ax.set_title("RMSE"); ax.legend()

plt.tight_layout()
plt.savefig("fig1_model_comparison.png", dpi=150, bbox_inches="tight", facecolor="#0f1117")
plt.close()
print("  Saved fig1_model_comparison.png")


# ── 6. FIGURE 2 – Feature Importance ─────────────────────────────────────────
print("Generating Figure 2: Feature Importance …")
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 6))
fig.patch.set_facecolor("#0f1117")
fig.suptitle(f"Feature Importance – Permutation ({best_name})", fontsize=15, fontweight="bold", color="#e6edf3")

def plot_fi(ax, fi_series, title, color):
    labels = [FEATURE_LABELS[f] for f in fi_series.index]
    bars   = ax.barh(labels[::-1], fi_series.values[::-1], color=color, alpha=0.85)
    ax.set_title(title, fontsize=12)
    ax.set_xlabel("Mean Importance (R² decrease)")
    for bar, val in zip(bars, fi_series.values[::-1]):
        ax.text(val + 0.001, bar.get_y() + bar.get_height()/2,
                f"{val:.4f}", va="center", fontsize=8, color="#c9d1d9")

plot_fi(ax1, fi_sales,  "→ Monthly Sales",  ACCENT)
plot_fi(ax2, fi_profit, "→ Monthly Profit", GREEN)

plt.tight_layout()
plt.savefig("fig2_feature_importance.png", dpi=150, bbox_inches="tight", facecolor="#0f1117")
plt.close()
print("  Saved fig2_feature_importance.png")


# ── 7. FIGURE 3 – Actual vs Predicted ────────────────────────────────────────
print("Generating Figure 3: Actual vs Predicted …")
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 6))
fig.patch.set_facecolor("#0f1117")
fig.suptitle(f"Actual vs Predicted ({best_name})", fontsize=15, fontweight="bold", color="#e6edf3")

def scatter_avp(ax, actual, pred, label, color):
    ax.scatter(actual, pred, alpha=0.5, s=20, color=color)
    mn = min(actual.min(), pred.min())
    mx = max(actual.max(), pred.max())
    ax.plot([mn, mx], [mn, mx], "--", color="#8b949e", lw=1.5, label="Perfect fit")
    r2 = r2_score(actual, pred)
    ax.set_title(f"{label}  (R²={r2:.4f})", fontsize=12)
    ax.set_xlabel("Actual"); ax.set_ylabel("Predicted")
    ax.legend(fontsize=8)

scatter_avp(ax1, ys_test, best["sales"]["pred"],  "Monthly Sales",  ACCENT)
scatter_avp(ax2, yp_test, best["profit"]["pred"], "Monthly Profit", GREEN)

plt.tight_layout()
plt.savefig("fig3_actual_vs_predicted.png", dpi=150, bbox_inches="tight", facecolor="#0f1117")
plt.close()
print("  Saved fig3_actual_vs_predicted.png")


# ── 8. FIGURE 4 – Business Insights ──────────────────────────────────────────
print("Generating Figure 4: Business Insights …")
fig = plt.figure(figsize=(18, 12))
fig.patch.set_facecolor("#0f1117")
gs = gridspec.GridSpec(2, 3, figure=fig, hspace=0.45, wspace=0.35)

# 4a – Avg Sales by Store Type
ax = fig.add_subplot(gs[0, 0])
cols = [ACCENT, GREEN, ORANGE, PURPLE]
bars = ax.bar(by_type.index, by_type["monthly_sales"] / 1000, color=cols, alpha=0.87)
ax.set_title("Avg Monthly Sales by Store Type", fontsize=11)
ax.set_ylabel("Sales (£000s)")
for bar in bars:
    ax.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 3,
            f"£{bar.get_height():.0f}k", ha="center", fontsize=8, color="#e6edf3")

# 4b – Profit Margin by Store Type
ax = fig.add_subplot(gs[0, 1])
bars = ax.bar(margin_by_type.index, margin_by_type, color=cols, alpha=0.87)
ax.set_title("Avg Profit Margin % by Store Type", fontsize=11)
ax.set_ylabel("Profit Margin %")
for bar in bars:
    ax.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.2,
            f"{bar.get_height():.1f}%", ha="center", fontsize=8, color="#e6edf3")

# 4c – Sales & Profit by Region
ax = fig.add_subplot(gs[0, 2])
reg_cols = [ACCENT, GREEN, ORANGE, PURPLE]
x = np.arange(len(by_region))
w = 0.4
ax.bar(x - w/2, by_region["monthly_sales"] / 1000,  w, label="Sales",  color=ACCENT, alpha=0.85)
ax.bar(x + w/2, by_region["monthly_profit"] / 1000, w, label="Profit", color=GREEN,  alpha=0.85)
ax.set_xticks(x); ax.set_xticklabels(by_region.index)
ax.set_title("Sales & Profit by Region", fontsize=11)
ax.set_ylabel("Amount (£000s)"); ax.legend(fontsize=8)

# 4d – Marketing Spend vs Sales scatter
ax = fig.add_subplot(gs[1, 0])
for i, st in enumerate(df["store_type"].unique()):
    sub = df[df["store_type"] == st]
    ax.scatter(sub["marketing_spend"] / 1000, sub["monthly_sales"] / 1000,
               alpha=0.4, s=15, color=PALETTE[i], label=st)
ax.set_title("Marketing Spend vs Monthly Sales", fontsize=11)
ax.set_xlabel("Marketing Spend (£000s)"); ax.set_ylabel("Sales (£000s)")
ax.legend(fontsize=7)

# 4e – Footfall vs Profit
ax = fig.add_subplot(gs[1, 1])
for i, st in enumerate(df["store_type"].unique()):
    sub = df[df["store_type"] == st]
    ax.scatter(sub["footfall"], sub["monthly_profit"] / 1000,
               alpha=0.4, s=15, color=PALETTE[i], label=st)
ax.set_title("Footfall vs Monthly Profit", fontsize=11)
ax.set_xlabel("Monthly Footfall"); ax.set_ylabel("Profit (£000s)")
ax.legend(fontsize=7)

# 4f – Customer Rating vs Profit Margin
ax = fig.add_subplot(gs[1, 2])
pm = profit_margin.copy()
for i, st in enumerate(pm["store_type"].unique()):
    sub = pm[pm["store_type"] == st]
    ax.scatter(sub["customer_rating"], sub["profit_margin_pct"],
               alpha=0.4, s=15, color=PALETTE[i], label=st)
ax.set_title("Customer Rating vs Profit Margin %", fontsize=11)
ax.set_xlabel("Customer Rating"); ax.set_ylabel("Profit Margin %")
ax.legend(fontsize=7)

fig.suptitle("Business Intelligence Dashboard", fontsize=17, fontweight="bold",
             color="#e6edf3", y=1.01)
plt.savefig("fig4_business_insights.png", dpi=150, bbox_inches="tight", facecolor="#0f1117")
plt.close()
print("  Saved fig4_business_insights.png")


# ── 9. FIGURE 5 – Correlation Heatmap ────────────────────────────────────────
print("Generating Figure 5: Correlation Heatmap …")
numeric_cols = ["marketing_spend","footfall","avg_discount_pct","staff_count",
                "inventory_availability_pct","competitor_distance_km","holiday_flag",
                "customer_rating","monthly_sales","monthly_profit"]
corr = df[numeric_cols].corr()
labels = [FEATURE_LABELS.get(c, c) for c in numeric_cols]

fig, ax = plt.subplots(figsize=(12, 9))
fig.patch.set_facecolor("#0f1117")
cmap = sns.diverging_palette(220, 10, as_cmap=True)
sns.heatmap(corr, annot=True, fmt=".2f", cmap=cmap, center=0,
            xticklabels=labels, yticklabels=labels, ax=ax,
            linewidths=0.5, linecolor="#21262d",
            annot_kws={"size": 7.5},
            cbar_kws={"shrink": 0.8})
ax.set_title("Feature Correlation Heatmap", fontsize=14, fontweight="bold", color="#e6edf3", pad=15)
plt.xticks(rotation=35, ha="right", fontsize=8)
plt.yticks(rotation=0, fontsize=8)
plt.tight_layout()
plt.savefig("fig5_correlation_heatmap.png", dpi=150, bbox_inches="tight", facecolor="#0f1117")
plt.close()
print("  Saved fig5_correlation_heatmap.png")


# ── 10. Print Summary ─────────────────────────────────────────────────────────
print("\n" + "="*60)
print("REGRESSION ANALYSIS COMPLETE")
print("="*60)
print(f"\nBest Model: {best_name}")
print(f"  Sales  → R²={best['sales']['r2']:.4f}  RMSE=£{best['sales']['rmse']:,.0f}  MAE=£{best['sales']['mae']:,.0f}")
print(f"  Profit → R²={best['profit']['r2']:.4f}  RMSE=£{best['profit']['rmse']:,.0f}  MAE=£{best['profit']['mae']:,.0f}")

print("\nTop 3 Sales Drivers:")
for feat, val in fi_sales.head(3).items():
    print(f"  {FEATURE_LABELS[feat]:<30} importance={val:.4f}")

print("\nTop 3 Profit Drivers:")
for feat, val in fi_profit.head(3).items():
    print(f"  {FEATURE_LABELS[feat]:<30} importance={val:.4f}")

print("\nAvg Monthly Sales by Store Type:")
for st, row in by_type.iterrows():
    margin = margin_by_type[st]
    print(f"  {st:<14} Sales=£{row['monthly_sales']:>10,.0f}  Profit Margin={margin:.1f}%")

print("\nAvg Monthly Sales by Region:")
for reg, row in by_region.iterrows():
    print(f"  {reg:<8} Sales=£{row['monthly_sales']:>10,.0f}  Profit=£{row['monthly_profit']:>9,.0f}")

print("\nOutputs: fig1_model_comparison.png, fig2_feature_importance.png,")
print("         fig3_actual_vs_predicted.png, fig4_business_insights.png,")
print("         fig5_correlation_heatmap.png")
