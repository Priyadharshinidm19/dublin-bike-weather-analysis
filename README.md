# Understanding the Influence of Weather on Dublin Bike Usage

Analysis of how weather conditions affect Dublin Bikes usage patterns, with a focus on commute vs. non-commute hours. Built with Python (pandas, NumPy, Seaborn) for data processing and Power BI for the final explanatory visualisation.

## Overview

This project combines ~1.88 million Dublin Bikes station-status records (March–May 2025, 5-minute snapshots across 115+ stations) with hourly Met Éireann weather observations (Dublin Airport) to explore whether — and how much — weather changes cycling behaviour throughout the day.

**Key finding:** time of day, especially the morning and evening commute, is by far the strongest driver of bike usage. Weather has a surprisingly small effect — even heavy rain only slightly reduces usage, temperature effects are mild, and central Dublin stations stay busiest regardless of conditions.

## Repository Contents

| File | Description |
|---|---|
| `Dublin_BikesData_Python.ipynb` | Jupyter notebook covering data loading, cleaning, merging, feature engineering, and exploratory analysis |
| `Dublin_Bike_Usage_Weather_Analysis.pbix` | Power BI Desktop file containing the final interactive dashboard |
| `Dublin_Bike_Usage_Weather_Analysis.pdf` | PDF export of the Power BI dashboard |
| `Understanding_the_Influence_of_Weather_on_Dublin_Bike_Usage_Report.pdf` | Full written report: methodology, visual design rationale, findings, and conclusion |

## Data Sources

- **Dublin Bikes station status** (March–May 2025) — [Smart Dublin / dublinbikes API](https://data.smartdublin.ie/dataset/dublinbikes-api): ~1.88M rows of 5-minute snapshots (bike/dock availability, station ID, capacity, lat/lon, timestamps) across 115+ stations.
- **Met Éireann hourly weather observations**, Dublin Airport — [met.ie historical data](https://www.met.ie/climate/available-data/historical-data): 2,207 rows covering rainfall, temperature, wind speed, and relative humidity for the same period.

This qualifies as a big-data problem across three dimensions: **volume** (~2M bike records), **variety** (behavioural + meteorological data), and **velocity** (bikes updated every 5 minutes vs. weather hourly).

## Methodology

1. **Ingestion** — The three monthly `station_status` CSVs were appended in Power BI / Power Query; due to file size, DAX Studio was used to export the full appended table to a single CSV for Python.
2. **Cleaning (bikes)** — Parsed `last_reported` to datetime, dropped rows missing timestamps/station IDs, cast counts to numeric, removed invalid (negative) values, and dropped irrelevant columns (`region_id`, `short_name`).
3. **Cleaning (weather)** — Parsed dates, filtered to March–May 2025, kept only rain/temperature/wind speed/humidity, and coerced to numeric.
4. **Feature engineering**
   - `bikes_in_use`, `datetime_hour` (bike timestamps floored to the hour)
   - `hour_of_day`, `weekday`, `is_weekend`
   - `is_commute_hour` — weekday 7–9 AM and 4–7 PM
   - `rain_band` — no rain / light / moderate / heavy
   - `temp_band` — very cold / cold / mild / warm / hot
5. **Merge** — Left join of bike snapshots to hourly weather on `datetime_hour`; succeeded for 99.96% of rows (early-hour weather gaps account for the remainder). Result: ~1.88M merged rows, trimmed to the fields needed and exported (`bikes_weather_rowlevel_clean.csv`) for Power BI.
6. **Visualisation** — Built entirely in Power BI: one main explanatory chart plus three supporting visuals (see below).

## Dashboard

**Main visual — "How Weather and Commute Behaviour Influence Bike Usage in Dublin":** a small-multiples line chart (commute vs. non-commute) with one line per rain category, directly answering the research question.

**Supporting visuals:**
- Heatmap of average bikes-in-use by hour of day, split by rainfall level
- Bubble map of station usage across Dublin, coloured by rain band
- Bar chart of usage by temperature band vs. commute/non-commute

## Key Findings

- **Time of day dominates.** Usage dips during the morning/evening commute windows and rises outside them — this pattern holds across all rain categories.
- **Rain has a small effect.** Average bikes-in-use ranges only ~11.9–12.1 across no-rain/light/moderate/heavy rain; even heavy rain barely dents commute-hour cycling.
- **Temperature has a mild effect.** Usage is slightly higher on cold/very-cold days (~12.2–12.3 avg) and lowest on hot days (~11.5), but the gap is small relative to the time-of-day effect.
- **Central Dublin stations stay busiest** regardless of weather conditions.

## Tools

- **Python:** pandas, NumPy, Seaborn, Matplotlib (Jupyter Notebook)
- **Power BI Desktop** + Power Query, DAX Studio (for large CSV export)

## Limitations & Future Work

- No true trip-level data — station snapshots approximate usage rather than capturing individual trips.
- Analysis is limited to a single spring quarter (March–May 2025); extending across seasons/years would capture seasonal effects.
- Additional weather variables (wind chill, visibility) could be incorporated.

## Authors

- Priyadharshini Dhanaraj Muthamil Selvi
- Rajeshwari Tajnekar

Completed as coursework for CSC1143 Data Management and Visualisation, Dublin City University, School of Computing.

## References

- [Met Éireann — Historical Weather Data](https://www.met.ie/climate/available-data/historical-data)
- [Smart Dublin — Dublin Bikes API](https://data.smartdublin.ie/dataset/dublinbikes-api)
- [Weather and Cycling in Dublin (Irish Cycle)](https://irishcycle.com/wp-content/uploads/2016/11/WeatherandCyclinginDublin.pdf)
- Bean, R., Pojani, D., & Corcoran, J. (2021). *How does weather affect bikeshare use? A comparative analysis of forty cities across climate zones.* Journal of Transport Geography, 95, 103155. https://doi.org/10.1016/j.jtrangeo.2021.103155
