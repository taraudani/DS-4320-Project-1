```python
import pandas as pd
import numpy as np
import requests
import os
import logging

#reading in drought monitor csv

try:
    drought = pd.read_csv('drought_monitor.csv')
    drought['DATE']  = pd.to_datetime(drought['ValidStart'])
    drought['YEAR']  = drought['DATE'].dt.year
    drought['MONTH'] = drought['DATE'].dt.month

    #D columns are cumulative so convert to incremental slices first
    #e.g. D1 means "D1 or worse" so D1_only = D1 - D2
    drought['DROUGHT_INDEX'] = (
        (drought['D0'] - drought['D1']) * 1 +
        (drought['D1'] - drought['D2']) * 2 +
        (drought['D2'] - drought['D3']) * 3 +
        (drought['D3'] - drought['D4']) * 4 +
        (drought['D4'])                 * 5
    ) / 100  # scale 0-5

    #average weekly index up to monthly, one row per month
    drought_monthly = (
        drought.groupby(['YEAR', 'MONTH'])
        .agg(DROUGHT_INDEX = ('DROUGHT_INDEX', 'mean'))
        .reset_index()
    )

    #turning monthly values into parquet files (w/ logs)

    drought_monthly.to_parquet('data/drought_monitor.parquet', index=False)
    log.info(f'drought_monitor: {len(drought_monthly)} monthly rows saved')
    print(f'Table 4 loaded: {len(drought_monthly)} monthly drought records')
    drought_monthly.head()

except Exception as e:
    log.error(f'drought_monitor failed: {e}')
    raise

```
