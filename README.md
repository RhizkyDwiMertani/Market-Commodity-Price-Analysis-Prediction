# Requirement Document (RD)
## Data Science Project: Market Commodity Price Analysis & Prediction

| | |
|---|---|
| **Project Name** | Market Commodity Price Analysis & Prediction System |
| **Version** | 1.0 |
| **Date** | 8 September 2026 |
| **Prepared For** | Commodity Price Monitoring Initiative |
| **Sample Dataset Used** | `Beras.xlsx` (Rice / Beras — national + 34 provinces, Jan 2023–Jul 2026) |
| **Reusability** | Designed to work with ANY commodity file that follows the same column structure (rice, chili, onion, sugar, cooking oil, etc.) |

---

## 1. Background

Indonesia's food commodity price monitoring bodies (e.g., Kemendag / Bapanas / PIHPS-style reporting) regularly publish **monthly average retail prices per province** for staple commodities. Each commodity is distributed as a spreadsheet with an identical layout:

- One row per region (national average + 34 provinces)
- One column per month (wide/pivot format)
- Prices expressed in Rupiah (Rp), formatted as text with thousands separators, with `-` denoting missing data

Because every commodity file (rice, red chili, shallot, cooking oil, sugar, etc.) shares this exact structure, a **single reusable pipeline** can be built once and pointed at any commodity file to produce analysis and forecasts — without rewriting code for each commodity.

## 2. Problem Statement

Market price fluctuation is difficult to anticipate manually across 34 provinces and multiple commodities. Stakeholders (government price-stabilization teams, distributors, retailers, researchers) need a **data-driven, repeatable** way to:

1. Understand historical price trends and seasonality per region and nationally.
2. Detect anomalies / regions with abnormal price spikes.
3. Forecast future prices (short-term, e.g. next 1–6 months) to support policy or business decisions (e.g., stock releases, subsidies, distribution planning).

## 3. Objectives

| # | Objective |
|---|---|
| O1 | Build a generic ETL pipeline that ingests any commodity file in the standard wide format and reshapes it into an analysis-ready long (tidy) format. |
| O2 | Perform exploratory data analysis (EDA): national trend, provincial comparison, seasonality, missing-data assessment, volatility. |
| O3 | Engineer time-aware features (lag, rolling statistics, seasonal encodings, region encoding) suitable for both statistical and machine-learning models. |
| O4 | Build and compare at least two forecasting approaches: (a) classical time-series model, (b) machine-learning regression model using panel (cross-provincial) data. |
| O5 | Evaluate models with standard forecast-accuracy metrics and select the best approach. |
| O6 | Produce forward price forecasts (next N months) per province and nationally, with visualizations. |
| O7 | Package the whole workflow as a single, well-documented Jupyter Notebook that any team member can re-run on a **different commodity file** by only changing a configuration cell. |

## 4. Scope

### 4.1 In Scope
- Data cleaning & transformation (wide → long/tidy format)
- Handling missing values (`-`) and text-formatted numbers (thousands separators)
- Exploratory Data Analysis & visualization
- Feature engineering (lag features, rolling mean/std, calendar/seasonal features, region encoding)
- Time-series forecasting (e.g., Holt-Winters / SARIMA)
- Machine-learning regression forecasting (Random Forest, XGBoost) using panel data across all provinces
- Model evaluation (MAE, RMSE, MAPE) and comparison
- Multi-step-ahead forecast generation and visualization
- Documentation for adapting the notebook to other commodities

### 4.2 Out of Scope
- Real-time data ingestion / API integration
- Deployment as a production web service or dashboard (can be a future phase)
- External macroeconomic/weather/supply-chain data enrichment (mentioned only as a future improvement)
- Commodity-specific domain calibration (e.g., rice harvest-cycle rules) beyond generic seasonal features

## 5. Data Description

### 5.1 Source Format (generic — applies to any commodity)

| Column | Description | Example |
|---|---|---|
| `No` | Row index in Roman numerals | `I`, `II`, `III`, ... |
| `Komoditas (Rp)` / region name column | Name of the region — row 1 is always the national average ("Semua Provinsi"), followed by 34 provinces | `Semua Provinsi`, `Aceh`, `Sumatera Utara`, ... |
| One column per month | Monthly average price for that region, as text with thousands separator; `-` = missing | `12,650`, `-` |

- **Rows:** 35 (1 national aggregate + 34 provinces)
- **Columns:** 2 identifier columns + N month columns (N grows over time as new months are published)
- **Granularity:** Monthly, per province
- **Currency/Unit:** Indonesian Rupiah (Rp), typically per kilogram

### 5.2 Sample Statistics (Beras / Rice dataset)
- Time range: January 2023 – July 2026 (43 months)
- Regions: 34 provinces + national average
- Missing values present (marked `-`), primarily in early periods for a few provinces (e.g., Aceh, Sumatera Barat)

### 5.3 Target Variable
`Price (Rp)` per region per month — this is what the models will forecast.

## 6. Methodology

1. **Data Ingestion & Cleaning**
   - Load Excel file, detect the region column and all date columns dynamically (so the pipeline works regardless of how many months are present)
   - Convert `-` to `NaN`, strip thousands separators, cast to numeric
   - Parse date columns (format `dd/ mm/ yyyy` with irregular spacing) into proper `datetime`
   - Melt wide format into long/tidy format: `[Region, Date, Price]`

2. **Missing Value Handling**
   - Assess missing pattern per region
   - Impute via time-based interpolation per region (forward-fill / linear interpolation) where appropriate

3. **Exploratory Data Analysis**
   - National price trend over time
   - Provincial comparison (highest/lowest price regions)
   - Seasonal patterns (month-of-year effect)
   - Volatility / month-over-month % change
   - Missing-data heatmap

4. **Feature Engineering**
   - Calendar features: year, month, quarter, cyclical month encoding (sin/cos)
   - Lag features: price at t-1, t-2, t-3, t-12 (prior year same month)
   - Rolling statistics: 3-month and 6-month rolling mean/std
   - Region encoding: one-hot or target/label encoding for ML models
   - Month-over-month % change

5. **Train / Test Split**
   - Time-based split (not random) — e.g., last 3–6 months held out as test set, respecting temporal order per region

6. **Modeling**
   - **Baseline:** naive last-value forecast, moving average forecast
   - **Time-Series Model:** Holt-Winters Exponential Smoothing / SARIMA on the national ("Semua Provinsi") series and optionally per-province
   - **Machine-Learning Regression:** Random Forest Regressor and XGBoost Regressor trained on the panel (long-format, multi-province) dataset using engineered features

7. **Evaluation**
   - Metrics: MAE, RMSE, MAPE
   - Compare all models on the held-out test period
   - Visualize actual vs. predicted

8. **Forecasting**
   - Generate N-month-ahead forecasts (default 3 months) using the best-performing model
   - Visualize forecast with confidence context

9. **Generalization**
   - All commodity-specific values (file path, commodity name, unit) are isolated in a single configuration cell at the top of the notebook, so the same notebook can be re-run on a different commodity file (e.g., `CabaiMerah.xlsx`, `BawangMerah.xlsx`) by changing only that cell — provided the new file follows the same column structure described in Section 5.1.

## 7. Tools & Technology Stack

| Category | Tool |
|---|---|
| Language | Python 3 |
| Data manipulation | pandas, numpy |
| Visualization | matplotlib, seaborn |
| Time-series modeling | statsmodels (Holt-Winters, SARIMA) |
| Machine learning | scikit-learn (Random Forest), xgboost |
| Environment | Jupyter Notebook |
| Input format | `.xlsx` (Excel), same schema as `Beras.xlsx` |

## 8. Deliverables

1. `README.md` — this requirement document
2. `Market_Price_Prediction.ipynb` — end-to-end analysis & forecasting notebook (generic, reusable across commodities)
3. Cleaned long-format dataset (generated at runtime, optionally exportable to CSV)
4. Forecast output table & charts (generated at runtime)

## 9. Assumptions

- Every commodity file follows the exact wide-format schema described in Section 5.1 (region column + monthly Rp columns).
- Monthly granularity is sufficient for the intended use case (no daily/weekly forecasting required).
- Missing values are true gaps in reporting, not zero prices, and can be reasonably interpolated.
- Historical patterns (seasonality, trend) are informative for near-term (1–6 month) forecasting; the model is not expected to predict shocks from unforeseen events (e.g., sudden policy change, extreme weather, geopolitical disruption).

## 10. Risks & Limitations

| Risk | Impact | Mitigation |
|---|---|---|
| Missing data concentrated in specific provinces/periods | Biased imputation | Use per-region interpolation; flag regions with high missing % |
| Short history (~3.5 years) limits ability to learn multi-year seasonal cycles | Lower forecast reliability for long horizons | Limit forecast horizon to short-term (1–6 months); prefer models that don't require long history |
| No external drivers (supply, weather, policy) in the data | Model blind to structural shocks | Document as a future enhancement (e.g., integrating rainfall, harvest calendar, fuel price) |
| Column count grows every month (new date column appended) | Hardcoded column ranges break | Pipeline detects date columns dynamically by regex pattern, not by fixed position |

## 11. Success Criteria

- Notebook runs end-to-end without manual intervention (besides the configuration cell) on both the sample rice dataset and at least one other commodity file with the same structure.
- Machine-learning model outperforms the naive baseline on MAPE by a clear margin on the held-out test set.
- Forecast and EDA visualizations are clear and interpretable for non-technical stakeholders.

## 12. Future Enhancements

- Incorporate external features: rainfall/climate data, fuel price, harvest calendar, regional supply/demand
- Build an interactive dashboard (e.g., Streamlit/Power BI) on top of the notebook's outputs
- Automate ingestion directly from the source portal (API) instead of manual Excel download
- Extend to multi-commodity joint modeling (cross-commodity price correlation)
