```python
import pandas as pd
import numpy as np
import requests
import os
import logging
import duckdb

#building a complete monthly calendar so every month appears even with no flood

try:
    years  = range(2015, 2025)
    months = range(1, 13)
    calendar = pd.DataFrame(
        [(y, m) for y in years for m in months],
        columns=['YEAR', 'MONTH']
    )

    #join all tables onto the calendar
    master = calendar.copy()
    master = master.merge(climate_monthly, on=['YEAR', 'MONTH'], how='left')
    master = master.merge(soil_monthly,    on=['YEAR', 'MONTH'], how='left')
    master = master.merge(drought_monthly, on=['YEAR', 'MONTH'], how='left')
    master = master.merge(
        flood_months[['YEAR', 'MONTH', 'FLOOD_COUNT', 'HAD_FLOOD']],
        on=['YEAR', 'MONTH'], how='left'
    )

    #fill months with no flood
    master['FLOOD_COUNT'] = master['FLOOD_COUNT'].fillna(0).astype(int)
    master['HAD_FLOOD']   = master['HAD_FLOOD'].fillna(0).astype(int)

    #lag-1 features so that prior month conditions predict next month floods
    master = master.sort_values(['YEAR', 'MONTH']).reset_index(drop=True)
    for col in ['PRCP_TOTAL', 'PRCP_MAX', 'SOIL_MOISTURE_MEAN',
                'SOIL_MOISTURE_MAX', 'DROUGHT_INDEX']:
        master[f'{col}_LAG1'] = master[col].shift(1)

    #turning into parquet file (w/ logs)
    master.to_parquet('data/master.parquet', index=False)
    master.to_csv('data/master.csv', index=False)
    log.info(f'master table: {master.shape} saved')
    print(f'Master table: {master.shape[0]} rows x {master.shape[1]} columns')
    print(f'Flood months: {master["HAD_FLOOD"].sum()} / {len(master)} total months')
    master.head()

except Exception as e:
    log.error(f'master build failed: {e}')
    raise

#connect to in-memory DuckDB database
con = duckdb.connect()

#load all parquet files directly into DuckDB tables
con.execute("CREATE TABLE weather_events  AS SELECT * FROM read_parquet('data/weather_events.parquet')")
con.execute("CREATE TABLE climate_monthly AS SELECT * FROM read_parquet('data/climate_monthly.parquet')")
con.execute("CREATE TABLE soil_moisture   AS SELECT * FROM read_parquet('data/soil_moisture_monthly.parquet')")
con.execute("CREATE TABLE drought_monitor AS SELECT * FROM read_parquet('data/drought_monitor.parquet')")
con.execute("CREATE TABLE master          AS SELECT * FROM read_parquet('data/master.parquet')")

#confirming all tables loaded
print('Tables loaded into DuckDB:')
print(con.execute("SHOW TABLES").df().to_string(index=False))
```
