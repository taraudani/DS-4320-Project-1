```python
import pandas as pd
import numpy as np
import requests
import os
import logging
import matplotlib.pyplot as plt

# setting up logging
os.makedirs('logs', exist_ok=True)
os.makedirs('data', exist_ok=True)
logging.basicConfig(
    filename='logs/pipeline.log',
    level=logging.INFO,
    format='%(asctime)s %(levelname)s %(message)s'
)
log = logging.getLogger(__name__)
print('Setup complete.')

# creating soil data by fetching from a free api
try:
    # Open-Meteo historical API
    url = (
        'https://archive-api.open-meteo.com/v1/archive'
        '?latitude=38.02&longitude=-78.48'
        '&start_date=2015-01-01&end_date=2025-12-31'
        '&daily=soil_moisture_0_to_7cm_mean'
        '&timezone=America%2FNew_York'
    )
    resp = requests.get(url, timeout=30)
    resp.raise_for_status()
    data = resp.json()

    # creating daily soil moisture dataframe
    soil_daily = pd.DataFrame({
        'DATE':          data['daily']['time'],
        'SOIL_MOISTURE': data['daily']['soil_moisture_0_to_7cm_mean']
    })
    soil_daily['DATE']          = pd.to_datetime(soil_daily['DATE'])
    soil_daily['YEAR']          = soil_daily['DATE'].dt.year
    soil_daily['MONTH']         = soil_daily['DATE'].dt.month
    soil_daily['SOIL_MOISTURE'] = pd.to_numeric(soil_daily['SOIL_MOISTURE'], errors='coerce')

    # aggregate to monthly soil moisture
    soil_monthly = soil_daily.groupby(['YEAR', 'MONTH']).agg(
        SOIL_MOISTURE_MEAN = ('SOIL_MOISTURE', 'mean'),
        SOIL_MOISTURE_MAX  = ('SOIL_MOISTURE', 'max'),
        SOIL_MOISTURE_MIN  = ('SOIL_MOISTURE', 'min')
    ).reset_index()

    soil_monthly.to_parquet('data/soil_moisture_monthly.parquet', index=False)
    log.info(f'soil_moisture: {len(soil_daily)} daily rows saved')
    print(f'Table 3 loaded: {len(soil_daily)} days of soil moisture data')
    soil_monthly.head()

except Exception as e:
    log.error(f'soil_moisture failed: {e}')
    raise
```
