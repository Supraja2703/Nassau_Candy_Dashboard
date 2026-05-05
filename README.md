# Nassau Candy Distribution Intelligence Platform

**Enterprise Decision Intelligence for Factory Reallocation & Shipping Optimization**

---

## Overview

A production-grade analytics platform that transforms raw distribution data into boardroom-ready decision intelligence. Built for Nassau Candy Distributor, this platform identifies factory-region routing inefficiencies, predicts lead times with ML-validated confidence, and surfaces prioritized recommendations for logistics optimization.

**Live capabilities:**
- Identifies addressable transport inefficiency in real time
- Provides ML-validated confidence intervals on recommendations
- Surfaces the top 3 prioritized actions on every page load
- Adapts all KPIs and alerts dynamically to sidebar filters

---

## Architecture

```
nassau_candy_dashboard.py   — Single-file Streamlit application
├── Design System           — Token-driven CSS, WCAG AA compliant
├── ML Pipeline             — Linear / RF / GBM with 5-fold CV + Bootstrap CI
├── Data Engine             — Real transport cost model (distance × rate × mode)
├── Executive Layer         — Alert bar, action strip, narrative generator
└── Analytics Tabs          — 9 analytical views (Factory, Scenario, Risk, SHAP, Map, ...)
```

---

## ML Pipeline

| Stage | Method | Output |
|---|---|---|
| Feature engineering | Haversine distance, seasonal flags, geo clustering | 15+ features |
| Model training | LinearRegression, RandomForest, GradientBoosting | 3 candidates |
| Selection | 5-fold CV RMSE (lowest = winner) | Best model |
| Confidence | Bootstrap resampling (n=80) | 90% CI on predictions |
| Explainability | SHAP values | Feature importance narrative |

---

## Key Design Decisions

**Transport cost model** — Distance × factory cost-per-km × ship mode multiplier × unit scaling. Rates are proxied from logistics benchmarks. Production use requires carrier contract data.

**Quantile-bounded heatmap** — zmin/zmax set at 5th/95th percentiles to prevent outlier-driven color range collapse. Worst-performing cell receives a visual annotation.

**Network Health Score** — Composite 0-100 metric: 40% lead time performance, 35% cost efficiency, 25% route risk. Gives executives one number to track across sessions.

**Bootstrap CI** — 80 resamples of the test set. Reports 90% CI. Does not account for distribution shift — appropriate for stable historical data, not real-time streaming.

**Seasonal features** — Holiday months defined as Oct/Nov/Dec/Feb for confectionery industry. Adjust `IS_HOLIDAY` logic in data preparation for company-specific peak periods.

---

## How to Run

```bash
# 1. Install dependencies
pip install streamlit pandas numpy plotly scikit-learn shap

# 2. Place your data file
cp your_data.csv Nassau_Candy_distributor_cleaned_data.csv

# 3. Launch
streamlit run nassau_candy_dashboard.py
```

Default credentials (change before deployment):
- `admin` / `admin123` — Administrator
- `analyst` / `analyst456` — Analyst
- `viewer` / `view789` — Viewer

---

## Data Dictionary

| Column | Type | Description |
|---|---|---|
| Order Date | date | Order placement date |
| Ship Date | date | Shipment dispatch date |
| Lead Time | int | Days from order to ship |
| Ship Mode | str | Standard / Second / First / Same Day |
| Factory | str | Source factory name |
| Region | str | Destination region |
| Division | str | Product category |
| Product Name | str | SKU name |
| Sales | float | Revenue per order |
| Units | int | Quantity shipped |
| Gross Profit | float | Revenue minus COGS |
| Cost | float | COGS |
| Net Margin | float | (Gross Profit - Transport Cost) / Sales |
| Transport Cost | float | Modeled logistics cost |
| Geo Distance KM | float | Haversine factory-to-region distance |
