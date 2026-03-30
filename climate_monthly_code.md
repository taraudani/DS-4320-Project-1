```python
import pandas as pd
import numpy as np
import requests
import os
import logging

#reading in climate history csv
try:
    clim = pd.read_csv('climate_history.csv', low_memory=False)
    clim['DATE']  = pd.to_datetime(clim['DATE'])
    clim['YEAR']  = clim['DATE'].dt.year
    clim['MONTH'] = clim['DATE'].dt.month

    #airport station has the longest complete record
    clim = clim[clim['NAME'] == 'CHARLOTTESVILLE ALBEMARLE AIRPORT, VA US'].copy()

    #coerce all numeric columns
    for col in ['TMAX', 'TMIN', 'PRCP', 'SNOW', 'AWND', 'WSF5']:
        clim[col] = pd.to_numeric(clim[col], errors='coerce')

    #aggregate to monthly values (mean, sum, or max value)
    climate_monthly = clim.groupby(['YEAR', 'MONTH']).agg(
        TMAX_MEAN       = ('TMAX', 'mean'),
        TMIN_MEAN       = ('TMIN', 'mean'),
        PRCP_TOTAL      = ('PRCP', 'sum'),
        PRCP_MAX        = ('PRCP', 'max'),        #largest single-day rain event
        HEAVY_RAIN_DAYS = ('PRCP', lambda x: (x > 1.0).sum()),  #days with >1 inch
        SNOW_TOTAL      = ('SNOW', 'sum'),
        WIND_MAX        = ('WSF5', 'max')
    ).reset_index()

    #creating monthly climate parquet file (w/ logs)
    climate_monthly.to_parquet('data/climate_monthly.parquet', index=False)
    log.info(f'climate_monthly: {len(climate_monthly)} monthly rows saved')
    print(f'Table 2 loaded: {len(climate_monthly)} months of climate data')
    climate_monthly.head()

except Exception as e:
    log.error(f'climate_monthly failed: {e}')
    raise
```
