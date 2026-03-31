# DS-4320-Project-1: Forecasting Monthly Flooding

### Executive Summary 
This repository contains the data, pipeline, and analysis for a machine learning project predicting flood events in Albemarle County, Virginia. Using four publicly available data sources on drought, soil moisture, precipitation conditions, and extreme weather events, a monthly dataset spanning 2015 through 2025 was assembled and stored using the relational model. A random forest classifier was trained on prior-month environmental conditions to predict whether a flood event would occur in Albemarle County in the following month, with soil moisture and precipitation emerging as the strongest predictors. All data is stored in parquet format and queryable via DuckDB, and the full pipeline from raw data ingestion through modeling and visualization is documented in the pipeline notebook.

**Name** - Tara Udani

**NetID** - hav7tz

**DOI** - [![DOI](https://zenodo.org/badge/1194739145.svg)](https://doi.org/10.5281/zenodo.19359764)

**Press release** - [Link to Press Release](https://github.com/taraudani/DS-4320-Project-1/blob/150f699b33bb813ba595ab1f8b79c02962994255/press_release.md)

**Data** - [Link to data folder](https://myuva-my.sharepoint.com/:f:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Data?csf=1&web=1&e=7KP18d)

**Pipeline** - [Link to Pipeline Folder](https://github.com/taraudani/DS-4320-Project-1/tree/150f699b33bb813ba595ab1f8b79c02962994255/pipeline)

**License** - [Link to License](https://github.com/taraudani/DS-4320-Project-1/blob/150f699b33bb813ba595ab1f8b79c02962994255/LICENSE)

## Problem Definition

### General and Specific Problem:
 - **General Problem:** Forecasting extreme weather events is a challenge and a necessity for local governments that must make emergency plans and  public safety responses in advance.

- **Specific Problem:** Using historical environmental monitoring data from 2015-2025 for Albemarle County, Virginia, which includes daily climate observations, antecedent soil moisture, drought conditions, and river streamflow, can we predict whether a flooding event will occur in a given month?

### Rationale:
Narrowing from the broad problem of extreme weather forecasting down to flooding specifically in Albemarle County is motivated by both data availability and clarity of causation. Flooding has a fairly clear chain of events that lead up to it: antecedent soil saturation, precipitation intensity, and watershed drainage capacity combined can determine whether a rain event becomes a flood. This means that predictors can be chosen due to known flooding processes rather than just correlation. Specifying Albemarle County was done because it is specifically impactful to the University of Virginia, and it also sits at the headwaters of the Rivanna River watershed, so the county experiences a range of flood-producing conditions, and has consistent environmental monitoring data across climate, soil moisture, and drought domains going back to 2015.

### Motivation:
The motivation for this project comes from the gap between raw environmental monitoring data and actionable flood forecasts at the local county level. Albemarle County has experienced 143 flood and flash flood events between 2015 and 2024, including 41 events in 2018 and 31 in 2020 alone, yet most residents and local planners still rely on the National Weather Service alerts, which provide just hours of warning. By building a monthly-scale predictive model that is grounded in historical patterns of environmental conditions, this project creates a predictive scale that allows citizens to be warned of potential flooding weeks in advance.  The result is a framework that any similarly monitored county could apply to improve its seasonal flood preparedness.

### Press Release Headline:
[The Flood Risk is Written in the Soil - How Albemarle County Environmental Data Predicts Flooding](https://github.com/taraudani/DS-4320-Project-1/blob/bf8d3c5e2a2b729a2858000829b5a690c6417128/press_release.md)

## Domain Exposition

### Terminology:
| Term | Definition |
|---|---|
| Flash Flood | A rapid flood event occurring within 6 hours of heavy rainfall, typically in small watersheds. |
| Antecedent Moisture | Soil water content in the days or weeks before a precipitation event. High antecedent moisture dramatically increases flood risk. |
| Soil Moisture (m³/m³) | Volumetric water content of soil, measured as cubic meters of water per cubic meter of soil. Values near 0.4 indicate near full saturation. |
| Drought Monitor (D0-D4) | A weekly categorical classification of drought intensity from D0 (abnormally dry) to D4 (exceptional drought), expressed as percent of county affected. |
| PRCP | Total precipitation in inches at a weather station. |
| WSF5 | Fastest 5-second wind gust recorded at a station on a given day |
| Flood Stage | The river gauge height at which flooding of nearby land begins. |
| ERA5 Reanalysis | A global climate reanalysis dataset produced by ECMWF that reconstructs historical weather conditions using models and observations. |

### Domain:
This project sits within the domain of hydrometeorological forecasting, which is a subfield of environmental data science that is concerned with predicting water-related weather events from atmospheric and land-surface conditions. The specific geographic focus is Albemarle County, Virginia, which drains into the Rivanna River. The county's topography, which is situated at the foot of the Blue Ridge Mountains, creates conditions where rainfall can quickly saturate small watersheds and produce flash flooding in low-lying areas of the county. The four data domains used in this project are daily climate, soil moisture, drought status, and flood event records, which are well-understood and correlated factors to flooding potential.

### Background Reading:
 [Link to OneDrive Folder with Readings](https://myuva-my.sharepoint.com/:f:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Articles?csf=1&web=1&e=nx3uJs)

### Reading Summary:
| # | Title | Description | Link |
|---|---|---|---|
| 1 | Severe Weather 101 - Flood Forecasting | An overview of how the National Weather Service uses antecedent soil moisture and precipitation thresholds to forecast flash flooding. | [Link](https://myuva-my.sharepoint.com/:b:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Articles/Severe%20Weather%20101_%20Flood%20Forecasting.pdf?csf=1&web=1&e=7ZTu5y) |
| 2 | Drought Classification | Explains how the D0-D4 classifications are produced and what data sources feed into them. | [Link](https://myuva-my.sharepoint.com/:b:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Articles/Drought%20Classification%20_%20U.S.%20Drought%20Monitor.pdf?csf=1&web=1&e=hqWJyM) |
| 3 | About the Data and Analysis |  Documentation explaining how ERA5 soil moisture reanalysis data is produced and validated. | [Link](https://myuva-my.sharepoint.com/:b:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Articles/About%20the%20data%20and%20analysis%20_%20Copernicus.pdf?csf=1&web=1&e=dencQg) |
| 4 | Local Effects of Climate Change | A report examining how climate change manifests locally through shifts in precipitation and extreme events, supporting the case for a Charlottesville-area flood analysis. | [Link](https://myuva-my.sharepoint.com/:b:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Articles/Local+Effects+of+Climate+Change%20(1).pdf?csf=1&web=1&e=3hdpdU) |
| 5 | Isotope hydrology and geophysical techniques for reviving a part of the drought prone areas of Vidarbha, Maharashtra, India | A peer-reviewed study on the relationship between antecedent soil moisture and flood magnitude in small watersheds. | [Link](https://myuva-my.sharepoint.com/:b:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Articles/Isotope%20hydrology%20and%20geophysical%20techniques%20for%20reviving%20a%20part%20of%20the%20drought%20prone%20areas%20of%20Vidarbha,%20Maharashtra,%20India%20-%20ScienceDirect.pdf?csf=1&web=1&e=cfLNU6) |

## Data Creation

### Provenance:
Four data sources were assembled to build the analytical dataset. The flood event records come from NOAA's Storm Events Database, filtered to Albemarle County extreme weather events, and then flood and flash flood events from 2015-2025. Daily climate observations were obtained from NOAA's Global Historical Climatology Network (GHCN) for the Charlottesville Albemarle Airport station which has the longest and most complete record in the area. Daily soil moisture data (0-7cm depth) was retrieved from the Open-Meteo historical weather API, which provides ERA5 reanalysis-based soil moisture estimates for Albemarle County with no account or API key required. Weekly drought classification data was downloaded from the US Drought Monitor for Albemarle County covering the full 2015-2025 period. All four sources are publicly available at no cost, require no special access permissions, and are updated on a regular cadence, which makes this dataset reproducible beyond the current study period.

### Code:
| File | Description | Link |
|---|---|---|
| climate_monthly| Describes the monthly mean/sum/max values of general climate variables such as temperature and precipitation | [Link](https://github.com/taraudani/DS-4320-Project-1/blob/a5599e7a1256e62b5a2c18ecf4ba0f8fd3ac6bd8/data%20code/climate_monthly_code.md) |
| drought_monitor| Monthly average index value for drought severity, ranging 0-5 (generally within 0-3) | [Link](https://github.com/taraudani/DS-4320-Project-1/blob/a5599e7a1256e62b5a2c18ecf4ba0f8fd3ac6bd8/data%20code/drought_monitor_code.md) |
| soil_moisture | Gives soil mosture mean value per month | [Link](https://github.com/taraudani/DS-4320-Project-1/blob/a5599e7a1256e62b5a2c18ecf4ba0f8fd3ac6bd8/data%20code/soil_moisture_code.md) |
| weather_events | The main predictive factor, which contains the occurrence of flooding by date and aggregates to number/occurrence by month in master sheet | [Link](https://github.com/taraudani/DS-4320-Project-1/blob/a5599e7a1256e62b5a2c18ecf4ba0f8fd3ac6bd8/data%20code/weather_events_code.md) |
| master | Contains all necessary variables from the above tables in one place for analysis, including DuckDB code used to query the relational database | [Link](https://github.com/taraudani/DS-4320-Project-1/blob/a5599e7a1256e62b5a2c18ecf4ba0f8fd3ac6bd8/data%20code/master_code.md) |

### Bias Identification:
Several sources of bias are present in this dataset. The climate data comes from a single station at the Charlottesville airport, which may not fully represent conditions across all of Albemarle County, as airport microclimates tend to be slightly warmer and drier than surrounding rural and forested areas. The flood event records rely on reports from trained spotters, emergency managers, and automated systems, meaning smaller or more rural events may go unreported while high-damage events near populated areas are more likely to be recorded. The soil moisture data is derived from the ERA5 reanalysis model rather than direct measurement, so they reflect modeled rather than observed conditions and may smooth over localized saturation. The drought monitor classifications are produced by a panel of experts synthesizing multiple data sources and may introduce subjectivity and human error or misjudgement.

### Bias Mitigation:
Spatial bias from single-station climate measurement is partially addressed by acknowledging that the analysis represents conditions at or near the airport rather than uniformly across the county, and by supplementing with the county-wide drought monitor classifications. Reporting bias in the flood event data can be partially quantified by comparing monthly event counts to long-term regional averages and flagging anomalously low-count years. ERA5 soil moisture bias can be assessed by comparing modeled values against known high-precipitation events to verify that the model captures saturation conditions correctly. For all numerical features, uncertainty is quantified using standard deviation statistics reported in the data dictionary.

### Rationale:
All data were aggregated to the monthly level to enable joining across four sources with different observation frequencies: daily climate and soil moisture, weekly drought classifications, and event-based flood records. Monthly aggregation loses some day-level detail but produces a complete, joinable analytical table. The choice to use lag-1 features (prior month's conditions) as predictors preserves the causal direction of the model (we predict next month's flood risk from this month's environmental conditions) and prevents data leakage where the same-month precipitation would be both a cause of flooding and a predictor of it. Flood events were filtered to Albemarle County specifically because the Storm Events Database records county-level events separately from weather zone events, and Albemarle County records are more geographically precise.

## Metadata

### ERD:
<img width="2172" height="1658" alt="image" src="https://github.com/user-attachments/assets/fad54b4a-1bdf-4480-9450-9117aae383a9" />

### Data:
| Table | Description | File |
|---|---|---|
| weather_events | NOAA flood and flash flood events in Albemarle County 2015-2024 | [Link](https://myuva-my.sharepoint.com/:u:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Data%20(parquet)/weather_events.parquet?csf=1&web=1&e=61T6Cm) |
| climate_monthly | Monthly aggregated climate observations from Charlottesville airport | [Link](https://myuva-my.sharepoint.com/:u:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Data%20(parquet)/climate_monthly.parquet?csf=1&web=1&e=SudKug) |
| soil_moisture | Daily ERA5 soil moisture at Albemarle centroid, aggregated to monthly | [Link](https://myuva-my.sharepoint.com/:u:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Data%20(parquet)/soil_moisture_monthly.parquet?csf=1&web=1&e=aI6BCt) |
| drought_monitor | Weekly US Drought Monitor classifications for Albemarle County | [Link](https://myuva-my.sharepoint.com/:u:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Data%20(parquet)/drought_monitor.parquet?csf=1&web=1&e=GZIgqv) |
| master | Final joined analytical table used for modeling | [Link](https://myuva-my.sharepoint.com/:u:/r/personal/hav7tz_virginia_edu/Documents/DS%204320%20Project%201/Data%20(parquet)/master.parquet?csf=1&web=1&e=NwGVbj) |

### Data Dictionary:
| Table | Feature | Type | Description | Example | Uncertainty |
|---|---|---|---|---|---|
| weather_events | `EVENT_ID` | int | Unique NOAA Storm Events identifier | 594741 | None — administrative primary key |
| weather_events | `EVENT_TYPE` | str | Flood classification (Flood or Flash Flood) | Flash Flood | Classification depends on NWS reporter judgment |
| weather_events | `BEGIN_DATE` | date | Date the event began | 2018-07-22 | Small events may be reported with a delay of hours to days |
| weather_events | `YEAR` | int | Calendar year extracted from BEGIN_DATE | 2018 | None — derived from BEGIN_DATE |
| weather_events | `MONTH` | int | Calendar month extracted from BEGIN_DATE | 7 | None — derived from BEGIN_DATE |
| climate_monthly | `YEAR` | int | Calendar year | 2020 | None |
| climate_monthly | `MONTH` | int | Calendar month | 6 | None |
| climate_monthly | `TMAX_MEAN` | float | Mean monthly high temperature (°F) | 84.2 | ±0.5°F station error; single station may not represent full county |
| climate_monthly | `TMIN_MEAN` | float | Mean monthly low temperature (°F) | 61.3 | ±0.5°F station error; single station may not represent full county |
| climate_monthly | `PRCP_TOTAL` | float | Total monthly precipitation (inches) | 4.72 | ±0.01in gauge error; airport may underrepresent western county orographic rainfall |
| climate_monthly | `PRCP_MAX` | float | Largest single-day precipitation total in the month (inches) | 2.10 | ±0.01in gauge error; point measurement may miss localized convective maxima |
| climate_monthly | `HEAVY_RAIN_DAYS` | int | Days in the month with precipitation exceeding 1.0 inch | 3 | Threshold of 1.0 inch is operationally common but somewhat arbitrary |
| climate_monthly | `SNOW_TOTAL` | float | Total monthly snowfall (inches) | 3.5 | ±0.1in observer error; snowfall is spatially variable across the county |
| climate_monthly | `WIND_MAX` | float | Maximum 5-second wind gust in the month (mph) | 43.0 | ±1mph anemometer error; airport may not capture terrain-channeled gusts |
| soil_moisture | `YEAR` | int | Calendar year | 2020 | None |
| soil_moisture | `MONTH` | int | Calendar month | 6 | None |
| soil_moisture | `SOIL_MOISTURE_MEAN` | float | Mean daily volumetric soil moisture 0–7cm depth for the month (m³/m³) | 0.283 | ±0.04 m³/m³ ERA5 model RMSE; reanalysis may smooth localized saturation events |
| soil_moisture | `SOIL_MOISTURE_MAX` | float | Maximum daily soil moisture 0–7cm depth in the month (m³/m³) | 0.312 | ±0.04 m³/m³ ERA5 model RMSE; peak values may be underestimated by smoothing |
| soil_moisture | `SOIL_MOISTURE_MIN` | float | Minimum daily soil moisture 0–7cm depth in the month (m³/m³) | 0.255 | ±0.04 m³/m³ ERA5 model RMSE |
| drought_monitor | `YEAR` | int | Calendar year | 2022 | None |
| drought_monitor | `MONTH` | int | Calendar month | 8 | None |
| drought_monitor | `DROUGHT_INDEX` | float | Monthly mean weighted drought severity (0–5 scale). 0 = no drought; 5 = entire county in exceptional drought. Derived from US Drought Monitor D0–D4 classifications. | 1.24 | ±~0.5 index units due to expert panel subjectivity in weekly D-level classifications |
| master | `FLOOD_COUNT` | int | Number of NOAA flood/flash flood events in Albemarle County that month | 5 | Subject to reporting bias; rural or nighttime events may go unrecorded |
| master | `HAD_FLOOD` | int | Binary target variable: 1 if at least one flood event occurred that month, 0 otherwise. 38 of 120 months (31.7%) are positive. | 1 | Inherits reporting bias from FLOOD_COUNT |
| master | `*_LAG1 columns` | float | Prior-month values of PRCP_TOTAL, PRCP_MAX, SOIL_MOISTURE_MEAN, SOIL_MOISTURE_MAX, and DROUGHT_INDEX. Used as model features to prevent data leakage. | PRCP_TOTAL_LAG1 = 3.21 | Inherits from source feature |
