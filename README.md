# Building‑Level Electricity Analytics & Forecasting

A full end‑to‑end workflow that cleans raw interval‑meter data, enriches it with weather + metadata, fills gaps, engineers features, and trains a LightGBM model to predict hourly kWh. The repo also ships an interactive geospatial dashboard to explore building‑level energy usage.

## Project Overview

This project tackles the common problem of messy, multi‑year building electricity data.  
We:

- **Reshape & unify** raw CSVs (one column = one building) into a tidy format.  
- **Align** timestamps to UTC while retaining local time.  
- **Merge** facility metadata and site‑level weather.  
- **Detect & repair** zero‑value outages and short data gaps.  
- **Engineer** rich temporal, spatial, and building‑size features.  
- **Train** a gradient‑boosted tree (LightGBM) with group‑aware cross‑validation.  
- **Explain** drivers via feature importance.  
- **Visualise** consumption patterns on an interactive Folium map.

## Data sources

| Dataset            | Path                              | Key Columns                                                                    |
| ------------------ | --------------------------------- | ------------------------------------------------------------------------------ |
| Electricity meters | `data/meters/raw/electricity.csv` | `timestamp`, `building_id`, `meter_reading`                                    |
| Building metadata  | `data/metadata/metadata.csv`      | `building_id`, `timezone`, `site_id`, `lat`, `lng`, `sqm`, `primaryspaceusage` |
| Hourly weather     | `data/weather/weather.csv`        | `timestamp`, `site_id`, `airTemperature`                                       |

## Data Source

This project uses the open‑data **Building Data Genome Project 2 (BDG2)**  
<https://github.com/buds-lab/building-data-genome-project-2>  
provided by the Building Urban Data Science (BUDS) Lab.

> **Citation**  
> Miller C., Kathirgamanathan A., Picchetti B. *et al.*  
> **“The Building Data Genome Project 2: Energy meter data from the ASHRAE Great Energy Predictor III competition.”**  
> *Scientific Data* 7, 368 (2020).  
> <https://doi.org/10.1038/s41597-020-00712-x>

BDG2 is released under the **GNU General Public License v3.0**. 

## Repository Structure

```
.
├── data/
│   ├── combinedmeters/
│   │   ├── meter_data_filtered.parquet
│   │   └── meter_with_temp.parquet
│   └── raw/
│       ├── electricity.csv
│       ├── metadata.csv
│       └── weather.csv
├── notebooks/
│   ├── building_energy_map.html
│   ├── data_preparation.ipynb
│   ├── load_forecasting.ipynb
│   └── smart_meter_data_combined.ipynb

```
## ETL & Data-Processing Pipeline

- Load & Melt `electricity.csv` → long format.
- Timezone Conversion per-building local → UTC (DST-safe).
- Merge building metadata from `metadata.csv` & air temperature from `weather.csv`.
- Output columnar Parquet (snappy) for fast downstream I/O.

## Data Quality & Gap Filling

-   **Zero-Gap Detection**: Contiguous 0 kWh blocks summarised; ≥24 h masked to NaN.
-   **Efficient Interpolation**: Linear fill of interior gaps ≤24 h (`interpolate_gaps_efficient`).
-   **Coverage Report**: Gap-length distribution & cumulative coverage.

## Exploratory Data Analysis

*   Building Size Distribution (`sqm`) – histogram & summary table.
*   Energy vs Size – boxplots (linear & log-scale).
*   Seasonal & work-hour averages summarised in the map popup.

![image](https://github.com/user-attachments/assets/360d2aaf-0dbb-4f36-84ba-a49caa5d24a1)

![image](https://github.com/user-attachments/assets/9f6bbe2e-fa21-487f-abae-8dd095efeaac)

## Feature Engineering

| Category          | Features                                                                                  |
| ----------------- | ----------------------------------------------------------------------------------------- |
| **Temporal**      | hour, day\_of\_week, month, is\_weekend, *cyclical* sine/cosine encodings                 |
| **Building**      | encoded IDs, `sqm`, `sqm_category`, `log_sqm`, `primaryspaceusage`, **energy\_intensity** |
| **Weather**       | `airTemperature`                                                                          |
| **Derived Flags** | weekend, working‑hours, season                                                            |


## Modelling • LightGBM

- 5-fold GroupKFold (group = building) to avoid leakage.
- MAE baseline: ≈ *30.4071 kWh* ± *4.5360*.
- Training time ≈ *14419.4* seconds(about 4 hours) on *14* CPU cores.

## Interactive Energy Map

`create_building_energy_map()` builds a Folium dashboard with:

* Clustered building markers sized & coloured by average kWh.
* Toggle layers for Winter / Spring / Summer / Fall consumption.
* Legend + search + fullscreen controls.


![Animation](https://github.com/user-attachments/assets/ba9337c9-4494-4875-92a8-ef895e371215)



