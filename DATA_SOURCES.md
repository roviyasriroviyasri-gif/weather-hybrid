# 🌐 Data Sources & Provenance Specification

## Overview
The Hybrid AI–NWP Weather Forecast Blending System for India consumes real operational NWP forecast model data and historical reanalysis datasets directly from Open-Meteo REST APIs.

---

## Supported External Data Sources

### 1. ECMWF High-Resolution Operational NWP Forecast
- **Provider**: European Centre for Medium-Range Weather Forecasts (ECMWF)
- **Model Endpoint**: `ecmwf_ifs025` (0.25° spatial resolution, ~25km grid)
- **Forecast Horizon**: 0 to 168 hours (Lead times: 6h, 12h, 18h, 24h, 36h, 48h, 72h, 96h, 120h, 144h, 168h)
- **API URL**: `https://api.open-meteo.com/v1/forecast`
- **Variables**:
  - `temperature_2m` (°C)
  - `precipitation` (mm)
  - `windspeed_10m` (m/s)
  - `relative_humidity_2m` (%)
  - `surface_pressure` (hPa)
- **Role in System**: Primary European NWP forecast candidate model.

---

### 2. GFS Global Forecast System NWP
- **Provider**: National Oceanic and Atmospheric Administration (NOAA) / NCEP
- **Model Endpoint**: `gfs_seamless`
- **Forecast Horizon**: 0 to 168 hours
- **API URL**: `https://api.open-meteo.com/v1/forecast`
- **Variables**:
  - `temperature_2m` (°C)
  - `precipitation` (mm)
  - `windspeed_10m` (m/s)
  - `relative_humidity_2m` (%)
  - `surface_pressure` (hPa)
- **Role in System**: Primary US NWP forecast candidate model.

---

### 3. ERA5 Reanalysis & Historical Observational Benchmark
- **Provider**: ECMWF ERA5 Climate Reanalysis
- **API URL**: `https://archive-api.open-meteo.com/v1/archive`
- **Date Range**: Historical observations spanning past 60 to 730 days
- **Variables**:
  - `temperature_2m` (°C)
  - `precipitation` (mm)
  - `windspeed_10m` (m/s)
  - `relative_humidity_2m` (%)
  - `surface_pressure` (hPa)
- **Role in System**: Ground truth target for AI base model training, out-of-sample skill score calculation, and historical verification. NOT treated as a live forecast source.

---

## Data Ingestion & Provenance Workflow
```
Live API Request / Scheduled Pipeline
                  ↓
   Data Ingestor (pipeline/ingest.py)
                  ↓
 Data Validator (pipeline/validation.py)
                  ↓
DuckDB Upsert Engine (storage/database.py)
                  ↓
Provenance Metadata Stamp (Source, Timestamp, Age, Status)
```

- **Source Status**:
  - `LIVE`: Data successfully fetched and validated within the last 24 hours.
  - `STALE`: Data is older than 24 hours.
  - `FETCH_ERROR`: API call failed or network offline.
