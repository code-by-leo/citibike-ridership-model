# Citi Bike Ridership Model

Predicting daily Citi Bike ridership in New York City from weather and calendar features, using BigQuery public data (50M+ trips joined to NOAA daily weather observations).

## Repository Structure

| Folder / File | Contents |
|---|---|
| `code/` | Jupyter notebooks: `eda.ipynb` (exploration), `preprocessing.ipynb` (cleaning), `model.ipynb` (regression model) — filled in Part 2 |
| `data/` | `citibike_weather_daily.csv` — the analysis-ready dataset (one row per day: rides, avg duration, weather) |
| `queries/` | `build_dataset.sql` — the BigQuery query that builds the dataset |
| `docs/` | Data dictionary, notes, and supporting material |

## Data Sources

- `bigquery-public-data.new_york_citibike.citibike_trips` — individual trip records (2013-07 to 2018-05)
- `bigquery-public-data.noaa_gsod.gsod20*` — daily weather from the LaGuardia Airport station (usaf 725030)
