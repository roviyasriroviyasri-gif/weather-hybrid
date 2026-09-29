# 🔍 Data Provenance, Quality & Integrity Specification

## Overview
Data provenance tracking ensures every forecast record and weather observation ingested into the blending system can be audited, verified, and traced back to its origin.

---

## Provenance Model Fields

Every record returned by the API or stored in DuckDB contains the following provenance metadata:

```json
{
  "data_source": "open_meteo_live_forecast",
  "last_updated_utc": "2026-09-15T14:00:00Z",
  "forecast_issue_time_utc": "2026-09-15T14:00:00Z",
  "valid_time_utc": "2026-09-16T14:00:00Z",
  "lead_time_hours": 24,
  "data_age_seconds": 120.5,
  "source_status": "LIVE"
}
```

- **`data_source`**: Identifies origin (`open_meteo_live_forecast`, `open_meteo_era5_archive`, `synthetic_test_fixture`).
- **`last_updated_utc`**: Precise UTC timestamp when data was fetched from external API.
- **`forecast_issue_time_utc`**: Model run initialization time (00:00 UTC, 06:00 UTC, 12:00 UTC, 18:00 UTC).
- **`valid_time_utc`**: Target forecast validity time in UTC.
- **`lead_time_hours`**: Forecast lead time ($\text{valid\_time} - \text{issue\_time}$).
- **`data_age_seconds`**: Elapsed time in seconds since data was initialized.
- **`source_status`**:
  - `LIVE`: Data fetched and validated within past 24 hours.
  - `STALE`: Data is older than 24 hours.
  - `FETCH_ERROR`: API call failed or network connection offline.

---

## Physical Data Validation Rules (`pipeline/validation.py`)

Every record is evaluated against physically defensible atmospheric bounds:

| Weather Element | Valid Physical Bounds | Action on Failure |
|---|---|---|
| **Latitude** | $6.0^\circ\text{N} \le \text{lat} \le 38.0^\circ\text{N}$ | Flagged Invalid |
| **Longitude** | $68.0^\circ\text{E} \le \text{lon} \le 98.0^\circ\text{E}$ | Flagged Invalid |
| **Temperature** | $-50.0^\circ\text{C} \le T \le 60.0^\circ\text{C}$ | Flagged Invalid |
| **Relative Humidity** | $0.0\% \le \text{RH} \le 100.0\%$ | Flagged Invalid |
| **Wind Speed** | $0.0 \le W \le 150.0\text{ m/s}$ | Flagged Invalid |
| **Precipitation** | $0.0 \le P \le 1000.0\text{ mm}$ | Flagged Invalid |
| **Surface Pressure** | $700.0 \le P_{\text{surf}} \le 1100.0\text{ hPa}$ | Flagged Invalid |

---

## Database Duplicate Prevention & Integrity Audit (`storage/database.py`)

1. **Unique Primary Keys**:
   - `forecasts`: `(issue_time_utc, lead_time_hours, location_id, model_name)`
   - `observations`: `(timestamp_utc, location_id, data_source)`
   - `blended_forecasts`: `(issue_time_utc, lead_time_hours, location_id, variable)`

2. **Deduplication Engine**: `StorageEngine.upsert_dataframe()` performs atomic staging and deduplication, preventing duplicate rows when API calls are re-executed.

3. **Integrity Audit Endpoint**: GET `/api/integrity` queries DuckDB database stats and reports total records, null counts, duplicate records, and health status.
