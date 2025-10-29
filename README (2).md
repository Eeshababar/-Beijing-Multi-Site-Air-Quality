# Beijing Air Quality — Time‑Series Analysis

A five‑notebook project for exploring and forecasting Beijing multi‑site air quality (focus: **PM2.5**) with meteorological covariates. Run the notebooks **top‑to‑bottom in the order shown** for a clean, reproducible workflow.

## 📁 Project structure

```
.
├── data/                           # Raw CSVs (Beijing multi-site AQ + weather)
├── notebooks/
│   ├── Time_series_data_analysis.ipynb        # 01 — EDA: loading, cleaning, visuals, quick insights
│   ├── Missing_Value_Handling.ipynb           # 02 — Gaps: alignment & imputation strategy
│   ├── Time_Series_Stationality_Test.ipynb    # 03 — (Stationarity) tests, transforms, differencing
│   ├── Time_Series_Decompose.ipynb            # 04 — STL/seasonal-trend decomposition & diagnostics
│   └── Advanced_data_analysis.ipynb           # 05 — Modeling & forecasts (ARIMA/SARIMAX, etc.)
├── results/                        # Saved figures, metrics, forecasts
├── requirements.txt
└── README.md
```

> **Note:** The filename uses “Stationality”; it *tests stationarity* (ADF/KPSS). Keep the filename as-is to match your repo.

## 🔧 Environment

Create a virtual environment and install dependencies:

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate    macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
```

**Minimal `requirements.txt` (edit as needed):**
```
pandas
numpy
matplotlib
plotly
seaborn
scipy
statsmodels
pmdarima
scikit-learn
missingno
prophet==1.1.5    # optional; pin if you use it
```
> If Prophet installation is troublesome, skip it; the core flow relies on statsmodels.

## 🧭 Notebook roles & expected outputs

### 01 — `Time_series_data_analysis.ipynb` (EDA)
**What it does**
- Load & harmonize multi‑site data; parse timestamps; unify units.
- Basic cleaning (duplicates, outliers), station normalization.
- Visuals: missingness overview, distributions, site comparisons, trends.
- Correlations: PM2.5 vs weather (TEMP, DEWP, PRES, RAIN, WSPM, wd).

**Saves**
- `results/eda/` figures (hist/box/violin; monthly/DOW/Hour profiles; correlation heatmap).
- Short EDA summary of drivers and seasonality hints.

### 02 — `Missing_Value_Handling.ipynb`
**What it does**
- Create a continuous **DatetimeIndex** at hourly cadence; align sensors/sites.
- Imputation strategies: forward/back fill within short windows, linear/time interpolation; optional seasonal Kalman/LOCF comparisons.
- Leakage checks and imputation quality diagnostics.

**Saves**
- `data/processed/` imputed datasets.
- `results/eda/missingness_*.png` and imputation comparison plots.

### 03 — `Time_Series_Stationality_Test.ipynb`
**What it does**
- Stationarity tests: **ADF**, **KPSS** (trend/level).
- Transformations: log(1+PM2.5), Yeo–Johnson (handles zeros/negatives).
- Differencing: regular and seasonal; order suggestion via ACF/PACF.
- Changepoint checks (optional).

**Saves**
- ACF/PACF plots, test tables → `results/figures/`

### 04 — `Time_Series_Decompose.ipynb`
**What it does**
- **STL** seasonal‑trend decomposition (e.g., 24‑hour, weekly).
- Seasonal strength & remainder diagnostics; seasonality stability across years.
- Site‑wise comparisons of seasonal components.

**Saves**
- STL component plots with confidence ribbons → `results/figures/stl_*`

### 05 — `Advanced_data_analysis.ipynb`
**What it does**
- Baselines: naïve, seasonal naïve, moving average.
- **ARIMA/SARIMA** (univariate PM2.5) and **SARIMAX/ARIMAX** with exogenous weather features.
- Time‑series cross‑validation (expanding/rolling windows), hyper‑parameter search (p,d,q)(P,D,Q)\_m.
- Residual diagnostics (Ljung–Box, remaining seasonality), forecast intervals.
- Optional: Prophet/TBATS or a small LSTM demo (for short horizon).

**Saves**
- `results/models/metrics_*.csv`, `forecasts_*.csv`
- Forecast vs actual plots with 80/95% PIs → `results/figures/`

## 🗂️ Data expectations

Place CSVs in `data/`. Typical columns include:
```
year, month, day, hour, station,
PM2.5, PM10, SO2, NO2, CO, O3,
TEMP, DEWP, PRES, RAIN, WSPM, wd
```
Make sure timestamps are parsed and set as index at **hourly** frequency.

**Minimal loader snippet**
```python
import pandas as pd
df = (pd.read_csv('data/Beijing_AirQuality.csv')
        .assign(ts=lambda d: pd.to_datetime(d[['year','month','day','hour']]))
        .set_index('ts').sort_index())
df = df.asfreq('H')  # enforce hourly grid
```

## ✅ Repro checklist

- [ ] EDA figures saved to `results/eda/`
- [ ] Imputed dataset in `data/processed/`
- [ ] Stationarity & STL diagnostics in `results/figures/`
- [ ] Model metrics table comparing baselines vs SARIMA/SARIMAX
- [ ] Final forecast plots & CSVs for selected horizons (e.g., 24h, 72h, 168h)

## 📈 Recommended figures for reports/slides
- Missingness heatmap (`missingno`)
- Seasonal boxplots (by month, DOW, hour)
- **STL** decomposition panels
- ACF/PACF with chosen orders annotated
- Forecast ribbons with prediction intervals
- Feature importance (if using exogenous models with tree/XGB)

## 📝 Notes & caveats
- PM2.5 is typically **right‑skewed** → consider log(1+PM2.5); back‑transform with bias correction for mean forecasts.
- Multi‑site: model stations separately **or** include station dummies/mixed effects.
- Avoid leakage: create splits chronologically and apply imputation/transformations **within** training folds when CV’ing.

## 📄 License & citation
- Respect the data source’s license (add formal citation for the Beijing multi‑site dataset here).
- Code in this repo: MIT (or update as appropriate).

---

**TL;DR:** Run the five notebooks in order (EDA → Missing → Stationarity → Decomposition → Modeling). Save figures/metrics to `results/` and processed data to `data/processed/` for easy reporting and reproducible forecasts.
