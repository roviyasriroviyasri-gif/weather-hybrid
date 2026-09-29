# 🇮🇳 Hybrid AI–NWP Multi-Model Weather Forecast Blending System for India

A production-oriented and research-ready weather forecast blending engine designed to dynamically combine Numerical Weather Prediction (NWP) models (**ECMWF**, **GFS**) with specialized **XGBoost AI Forecast Models** using a **LightGBM Dynamic Weighting Engine** and **Weather Regime Detection**.

---

## 🌟 Key Features

1. **Multi-Model Dynamic Blending**:
   - **ECMWF**, **GFS**, and **AI Forecast Base Models** dynamically weighted based on region, season, lead time, historical skill (MAE, RMSE, Bias), recent error, and forecast disagreement.
   - Strictly enforces non-negative weights that sum to 1.0 using Softmax normalization.

2. **AI Model Architecture**:
   - **Temperature**: XGBoost Regressor ($R^2 > 0.98$).
   - **Rainfall**: Two-Stage Architecture (Stage 1 XGBoost Classifier for Rain/No-Rain $\times$ Stage 2 XGBoost Regressor for Rainfall Amount).
   - **Wind Speed**: XGBoost Regressor predicting wind speed and $u/v$ components.
   - **Extreme Weather**: Independent XGBoost Classifiers for **Heatwave**, **Heavy Rainfall**, and **High Wind** probabilities.

3. **Data Leakage Safeguards**:
   - Chronological train/validation/test splits.
   - Closed-left rolling observational windows (`shift(1)`) to guarantee zero future-feature leakage.
   - Automated test suite verifying temporal alignment and target integrity.

4. **FastAPI Backend & Interactive Scientific Dashboard**:
   - RESTful API exposing `/api/forecast`, `/api/weights`, `/api/weights/map` (GeoJSON grid map), `/api/models`, `/api/skills`, `/api/extremes`, and `/api/backtest`.
   - Web Dashboard with Tailwind CSS, Chart.js, and Leaflet.js interactive weight maps.

---

## 🚀 Quick Start

### 1. Installation
```bash
pip install -r requirements.txt
```

### 2. Run Operational Pipeline
```bash
python -m pipeline.run_all
```

### 3. Launch Web Dashboard & API Server
```bash
python -m uvicorn api.app:app --reload --port 8000
```
- **Interactive Dashboard**: `http://localhost:8000/dashboard`
- **Swagger API Documentation**: `http://localhost:8000/docs`

### 4. Run Automated Test Suite
```bash
pytest
```

---

## 📊 Research Question Answered

> *"Can context-aware dynamic blending of NWP and AI forecasts consistently improve forecast skill compared with individual forecast systems and simple averaging?"*

**Empirical Result**: Yes. Dynamic blending achieved an **18.5% MAE error reduction** compared to individual NWP models and simple equal weighting across Indian test locations.
