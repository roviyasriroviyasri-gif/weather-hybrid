# 📡 REST API Reference Documentation

**Base URL**: `http://localhost:8000/api`  
**Interactive Swagger Docs**: `http://localhost:8000/docs`  
**ReDoc Documentation**: `http://localhost:8000/redoc`  

---

## Endpoints Summary

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | System health check and DB connectivity status |
| `GET` | `/api/forecast` | Retrieve blended multi-model forecasts (filterable by location, variable, lead time) |
| `GET` | `/api/forecast/temperature` | Retrieve temperature forecasts |
| `GET` | `/api/forecast/rainfall` | Retrieve rainfall forecasts |
| `GET` | `/api/forecast/wind` | Retrieve wind speed forecasts |
| `GET` | `/api/weights` | Retrieve dynamic weight breakdown summary |
| `GET` | `/api/weights/map` | Generate GeoJSON dynamic weight grid map for India |
| `GET` | `/api/models` | List trained AI base models, weighting model metadata, and metrics |
| `GET` | `/api/skills` | Retrieve historical forecast verification skill scores |
| `GET` | `/api/extremes` | Retrieve extreme weather event probabilities |
| `GET` | `/api/backtest` | Run out-of-sample backtesting comparison across models |
| `GET` | `/api/integrity` | Database audit report (total records, duplicates, nulls, status) |
| `GET` | `/dashboard` | Render 16-section interactive web dashboard |

---

## Detailed Endpoint Specifications

### 1. GET `/api/health`
**Response**:
```json
{
  "status": "healthy",
  "timestamp_utc": "2026-09-15T14:25:00Z",
  "version": "1.0.0",
  "database_connected": true,
  "data_source_status": "LIVE"
}
```

---

### 2. GET `/api/forecast`
**Query Parameters**:
- `location` (optional): `chennai`, `bengaluru`, `mumbai`, `delhi`, `kolkata`, `hyderabad`
- `variable` (optional): `temperature`, `rainfall`, `wind_speed`
- `lead_time` (optional): `24`, `48`, `72`, `120`, etc.

**Response**:
```json
[
  {
    "location_id": "chennai",
    "location_name": "Chennai",
    "latitude": 13.0827,
    "longitude": 80.2707,
    "issue_time_utc": "2026-09-15T14:00:00Z",
    "valid_time_utc": "2026-09-16T14:00:00Z",
    "lead_time_hours": 24,
    "variable": "temperature",
    "individual_forecasts": {
      "ecmwf": 31.2,
      "gfs": 32.1,
      "ai": 31.5
    },
    "weights": {
      "ecmwf": 0.25,
      "gfs": 0.20,
      "ai": 0.55
    },
    "blended_forecast": 31.55,
    "uncertainty_std": 0.38,
    "weather_regime": "MONSOON_ACTIVE",
    "extreme_guidance": {
      "heatwave_prob_pct": 12.0,
      "heavy_rainfall_prob_pct": 68.5,
      "high_wind_prob_pct": 24.0
    },
    "provenance": {
      "data_source": "open_meteo_live_forecast",
      "last_updated_utc": "2026-09-15T14:00:00Z",
      "forecast_issue_time_utc": "2026-09-15T14:00:00Z",
      "valid_time_utc": "2026-09-16T14:00:00Z",
      "lead_time_hours": 24,
      "data_age_seconds": 120.5,
      "source_status": "LIVE"
    },
    "explanation": "AI XGBoost received the highest dynamic weight (55.0%) for location Chennai...",
    "model_version": "v1.0.0"
  }
]
```

---

### 3. GET `/api/weights/map`
**Query Parameters**:
- `variable`: `temperature`, `rainfall`, `wind_speed`
- `lead_time`: `24`
- `season`: `SOUTHWEST_MONSOON`

**Response**: GeoJSON `FeatureCollection` with grid coordinates across India and dynamic model weight properties.
