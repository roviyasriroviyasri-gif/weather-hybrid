# 🤖 Model Architecture & Machine Learning Design

## Overview
The blending system combines Numerical Weather Prediction (NWP) outputs (**ECMWF**, **GFS**) with specialized **XGBoost AI Forecast Base Models** using a **LightGBM Dynamic Weighting Engine**.

---

## 1. AI Base Forecast Models (`models/base_models.py`)

### A. Temperature Model (XGBoost Regressor)
- **Target**: $2\text{m}$ Temperature ($^\circ\text{C}$)
- **Features**: `ecmwf_temperature_2m`, `gfs_temperature_2m`, `temp_ensemble_mean`, `temp_disagreement`, `latitude`, `longitude`, `elevation_m`, `hour`, `month`, `day_of_year`, `lead_time_hours`, `monsoon_indicator`, `rolling_temp_24h_mean`, `temp_change_3h`.
- **Validation Metrics**:
  - $R^2 = 0.989$
  - $\text{MAE} = 0.437^\circ\text{C}$
  - $\text{RMSE} = 0.609^\circ\text{C}$

---

### B. Rainfall Model (Two-Stage XGBoost Architecture)
- **Stage 1 (Occurrence Classifier)**:
  - **Model**: XGBoost Classifier (`scale_pos_weight` for class imbalance)
  - **Target**: Rain Event ($\text{Precipitation} > 0.1\text{ mm}$)
  - **Metrics**: $\text{ROC-AUC} = 0.869$, $\text{Precision} = 0.917$, $\text{F1} = 0.710$
- **Stage 2 (Amount Regressor)**:
  - **Model**: XGBoost Regressor trained on wet days
  - **Target**: Rainfall Amount ($\text{mm}$)
  - **Metrics**: $\text{Amount MAE} = 0.041\text{ mm}$
- **Combined AI Estimate**:
  $$\text{AI Rainfall} = P(\text{Rain}_{\text{Stage 1}}) \times \text{Amount}_{\text{Stage 2}}$$

---

### C. Wind Speed Model (XGBoost Regressor)
- **Target**: $10\text{m}$ Wind Speed ($\text{m/s}$)
- **Features**: NWP wind forecasts, spatial coordinates, elevation, rolling $24\text{h}$ max wind, temporal encodings.
- **Metrics**: $\text{MAE} = 0.522\text{ m/s}$, $\text{RMSE} = 0.693\text{ m/s}$.

---

## 2. Weather Regime Engine (`models/regime_engine.py`)
Classifies prevailing atmospheric state into 6 distinct Indian weather regimes:
1. `MONSOON_ACTIVE` (SW Monsoon rain/cloud, Jun-Sep)
2. `MONSOON_BREAK` (Low rain, high temp in Jun-Sep)
3. `PRE_MONSOON_HEAT` (Temp $\ge 38^\circ\text{C}$, Mar-May)
4. `POST_MONSOON_CYCLONIC` (High wind/rain, Oct-Nov)
5. `WINTER_DRY` (Temp $< 22^\circ\text{C}$, Dec-Feb)
6. `NORMAL_WESTERLY` (Default baseline)

---

## 3. LightGBM Dynamic Weighting Engine (`models/weighting_engine.py`)
- **Objective**: Predict inverse-error target weights for ECMWF, GFS, and AI.
- **Features**: `latitude`, `longitude`, `elevation_m`, `hour`, `month`, `lead_time_hours`, `weather_regime`, `forecast_disagreement`, `mae_ecmwf`, `mae_gfs`, `mae_ai`, `rmse_*`, `recent_error_*`.
- **Softmax Normalization**:
  $$w_m = \frac{\exp(\hat{s}_m)}{\sum_{k} \exp(\hat{s}_k)} \implies w_m \ge 0 \quad \text{and} \quad \sum w_m = 1.0$$

---

## 4. Forecast Fusion & Uncertainty (`models/fusion_engine.py`)
- **Linear Combination**:
  $$\text{Blended} = w_{\text{ecmwf}} \cdot \text{ECMWF} + w_{\text{gfs}} \cdot \text{GFS} + w_{\text{ai}} \cdot \text{AI}$$
- **Uncertainty Bounds**:
  $$\sigma_{\text{uncertainty}} = \sqrt{ w_{\text{ecmwf}}(\text{ECMWF} - \text{Blended})^2 + w_{\text{gfs}}(\text{GFS} - \text{Blended})^2 + w_{\text{ai}}(\text{AI} - \text{Blended})^2 }$$

---

## 5. Extreme Weather Classifiers (`models/extreme_engine.py`)
Three independent XGBoost binary classifiers outputting event probabilities:
- **Heatwave**: $\text{Temp} \ge 40^\circ\text{C}$ or anomaly $\ge 4.5^\circ\text{C}$.
- **Heavy Rainfall**: $\text{Precipitation} \ge 64.5\text{ mm/day}$.
- **High Wind**: $\text{Wind Speed} \ge 15.0\text{ m/s}$.
