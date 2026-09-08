# 🇮🇩 Food Commodity Price Analysis & Forecasting Across Indonesian Provinces (Reusable)

A generic data science portfolio project — one notebook that can analyze **any food commodity**
(rice, chili, onion, cooking oil, sugar, etc.) as long as the data follows the PIHPS/BPS
template (`No`, `Komoditas (Rp)`, followed by date columns).

## 📌 Problem Statement

Indonesia's food commodity prices are published province-by-province, month-by-month, in raw
spreadsheet form — not directly actionable for policy or analysis. Rice, the national staple, is
especially sensitive: price spikes affect inflation, purchasing power, and food security policy
(Bulog stock releases, market operations, import decisions).

This project turns that raw data into answers to concrete questions:
- Is the national price trending up or down, and how fast?
- Which provinces consistently pay more — and is the gap explained by known factors like
  inter-island logistics?
- Which provinces show the most volatile price swings, signaling possible supply instability?
- Can provinces be segmented (cheap / mid / expensive, fast-rising vs. stable) to prioritize
  stabilization efforts?
- What should we expect next month if no intervention happens?

The deliverable is a single, reusable notebook that answers these questions for any commodity
sharing the same data template — not just rice.

Tested and proven to work on 2 real-data scenarios:
- **Short dataset**: 7 months (Jan–Jul 2026)
- **Long historical dataset**: 43 months (Jan 2023–Jul 2026), including missing values (`-`)
  for some provinces/months

## ⚙️ How to Reuse for Another Commodity

Just change 2 lines in the configuration cell at the top of the notebook:

```python
FILE_PATH = 'data/your_commodity_file.xlsx'
COMMODITY_NAME = 'Chili'   # or 'Onion', 'Cooking Oil', etc.
```

Then **Run All**. All chart titles, labels, and output folders adapt automatically
(`output/<commodity_name>/...`).

## 🗂️ Project Structure

```
.
├── commodity_price_analysis.ipynb        # Main notebook (generic/reusable)
├── example_output_rice_historical.ipynb  # Example run on the 43-month historical dataset
├── data/
│   ├── rice.xlsx                         # Example data: 7 months
│   └── rice_historical.xlsx              # Example data: 43 months (with data gaps)
├── output/
│   ├── rice/                             # Charts from the 7-month analysis
│   └── rice_historical/                  # Charts from the 43-month analysis
└── README.md
```

## 🔍 Analysis Contents (applies to any commodity)

1. **Data Cleaning** — robust to short/long time ranges & missing values (`-` → NaN)
2. **National Trend** — average price movement over the observed period
3. **Regional Disparity** — most expensive vs. cheapest provinces in the latest month
4. **Volatility** — most stable vs. most volatile provinces (coefficient of variation)
5. **Segmentation (Clustering)** — K-Means based on price level & monthly trend
6. **Price Forecast** — linear regression per province for the next month

## 🛡️ Robustness Handled

- ✅ Flexible time range (tested on 7 months & 43 months)
- ✅ Missing values (`-`) in government data → automatically converted to `NaN`, no crashes
- ✅ Clustering & forecasting ignore months with missing data per province (instead of
  dropping the whole province row)
- ✅ Chart labels & output folder names automatically follow the chosen commodity name

## ⚠️ Limitations

- For short datasets (<12 months), seasonal patterns (Ramadan, harvest season) can't be captured
- Forecasting still uses simple linear regression — for 24+ month datasets like
  `rice_historical.xlsx`, seasonal methods (SARIMA/Prophet) would give more accurate results
  (see further development below)
- Only tested on one real commodity (rice) with two time-range variations; the template
  consistency assumption should be re-validated once real chili/onion/etc. data is available

## 🚀 Further Development

- With 43 months of data now available, add **seasonal decomposition** (`statsmodels`) to
  separate the long-term trend from annual seasonal patterns
- Compare multiple commodities in a single dashboard (Streamlit, with a commodity dropdown)
- Add exogenous variables: rainfall, farm-gate grain prices, exchange rate, fuel prices

## 🛠️ How to Run

```bash
pip install pandas numpy matplotlib scikit-learn openpyxl jupyter
jupyter notebook commodity_price_analysis.ipynb
```

## 🧰 Tools

`Python` · `pandas` · `numpy` · `matplotlib` · `scikit-learn` (K-Means, Linear Regression)
