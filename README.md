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

## Part 2 Write-Up: Predicting Daily Ridership

**The question:** can ops predict how many rides will happen on a given day using nothing but the calendar and a weather forecast?

**Data quality issues found.** The dataset looked clean and wasn't. First, `ride_date` loaded as a plain string rather than a datetime — CSVs carry no type information, so pandas left it as text; I converted it with `pd.to_datetime`. Second, `precip_in` had a maximum of 99.99 inches, which is NOAA's coded missing value for precipitation, not rain. Exactly one row (2016-02-11) was affected; I replaced the code with `NaN` and imputed 0.0, since the dataset's median precipitation is zero, ~63% of days are dry, and the neighboring days that week recorded 0.00–0.06 inches. Third, 186 calendar dates were missing entirely: Jan 23–26, 2016 is the Winter Storm Jonas shutdown (no ride rows — the rides side of Part 1's inner join), and Oct 2016–Mar 2017 is a six-month block with no weather observations (the weather side). The inner join silently dropped a day missing from either source.

**Features engineered.** Day of week was one-hot encoded (`drop_first=True`, so every coefficient reads relative to a Friday baseline). Because average July ridership roughly doubled from 2013 (~27k rides/day) to 2017 (~56k), the model needed to know what year it is: I built a `days_since_launch` trend feature, which captures smooth continuous growth better than a coarse year column. Per Rule 2, only mean temperature (`temp_f`) entered the model — the three temperature columns correlate at ~0.98 and stacking them just splits one effect across unstable coefficients. Per Rule 1, `avg_duration_min` was dropped in preprocessing since it's derived from the same day's rides. EDA also showed ridership rises with temperature but flattens past ~85 °F, so I added a squared temperature term.

**Performance.** The core model (temp, precipitation, wind, day-of-week, trend) reached test R² 0.744 with MAE ≈ 6,950 rides and RMSE ≈ 8,770 on a ~33,000-ride average day — typically off by about 7,000 rides, with no overfitting (train R² 0.732). Coefficients matched EDA intuition: each inch of rain costs ~3,900 rides, each degree of warmth adds ~570, Sundays run ~7,300 behind comparable Fridays. Residuals showed curvature, so the one improvement I made was adding `temp_sq`, lifting test R² to 0.753 by giving warm days diminishing returns.

**Biggest weakness.** The model's worst misses are calendar days that don't behave like their weather: holidays that ride like Sundays but are labeled weekdays, and snow days where modest liquid precipitation hides a blizzard. Next I'd want a US-holidays flag, GSOD's snowfall/snow-depth column, and monthly station counts to replace the linear trend with the system's actual size.
