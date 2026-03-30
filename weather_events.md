```python
import pandas as pd
import numpy as np
import requests
import os
import logging

#reading in weather events csv file 
try:
    events = pd.read_csv('weather_events.csv')
    events['BEGIN_DATE'] = pd.to_datetime(events['BEGIN_DATE'], format='%m/%d/%Y')
    events['YEAR']       = events['BEGIN_DATE'].dt.year
    events['MONTH']      = events['BEGIN_DATE'].dt.month

    #filter to only flood events in Albemarle County
    floods = events[
        (events['EVENT_TYPE'].isin(['Flood', 'Flash Flood'])) &
        (events['CZ_NAME_STR'] == 'ALBEMARLE CO.')
    ].copy()

    #only keeping necessary variables
    floods = floods[['EVENT_ID', 'EVENT_TYPE', 'BEGIN_DATE', 'YEAR', 'MONTH']]

    #aggregate to monthly counts 
    flood_months = (
        floods.groupby(['YEAR', 'MONTH'])
        .agg(FLOOD_COUNT = ('EVENT_ID', 'count'))
        .reset_index()
    )
    flood_months['HAD_FLOOD'] = 1 #binary count of if flood occurred in a month

    #converting to parquet file and saving (w/ logs)
    floods.to_parquet('data/weather_events.parquet', index=False)
    log.info(f'weather_events: {len(floods)} flood events saved')
    print(f'Table 1 loaded: {len(floods)} flood events')
    floods.head()

except Exception as e:
    log.error(f'weather_events failed: {e}')
    raise
```
