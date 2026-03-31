# **Pipeline**


```python
import pandas as pd
import numpy as np
import requests
import os
import logging
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
from sklearn.ensemble import RandomForestClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import cross_val_score, StratifiedKFold
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import classification_report, confusion_matrix, ConfusionMatrixDisplay
from sklearn.pipeline import Pipeline
import warnings
warnings.filterwarnings('ignore')
```

## DuckDB


```python
import duckdb
import pandas as pd

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

    Tables loaded into DuckDB:
               name
    climate_monthly
    drought_monitor
             master
      soil_moisture
     weather_events


## Database Queries


```python
#query 1: how many flood events occurred each year?
q1 = con.execute("""
    SELECT
        YEAR,
        COUNT(*)                    AS total_flood_events,
        COUNT(DISTINCT EVENT_TYPE)  AS event_types
    FROM weather_events
    GROUP BY YEAR
    ORDER BY YEAR
""").df()
print('Flood events by year:')
display(q1)
```

    Flood events by year:




  <div id="df-bfaf1c8f-689f-473a-b85d-3f978b146397" class="colab-df-container">
    <div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>YEAR</th>
      <th>total_flood_events</th>
      <th>event_types</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2015</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016</td>
      <td>7</td>
      <td>2</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2017</td>
      <td>3</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018</td>
      <td>41</td>
      <td>2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2019</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2020</td>
      <td>31</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2021</td>
      <td>7</td>
      <td>2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2022</td>
      <td>8</td>
      <td>2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2023</td>
      <td>6</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2024</td>
      <td>27</td>
      <td>2</td>
    </tr>
    <tr>
      <th>10</th>
      <td>2025</td>
      <td>6</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-bfaf1c8f-689f-473a-b85d-3f978b146397')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  
    }

```python
#query 2: which months are highest flood risk?
q2 = con.execute("""
    SELECT
        MONTH,
        SUM(HAD_FLOOD)                          AS flood_months,
        COUNT(*)                                AS total_months,
        ROUND(AVG(HAD_FLOOD) * 100, 1)          AS flood_pct,
        ROUND(AVG(PRCP_TOTAL), 2)               AS avg_precip_in,
        ROUND(AVG(SOIL_MOISTURE_MEAN), 3)       AS avg_soil_moisture
    FROM master
    GROUP BY MONTH
    ORDER BY MONTH
""").df()
print('Flood risk by month:')
display(q2)
```

    Flood risk by month:




  <div id="df-8807bbf8-95e7-4664-86d9-78feaab66bc7" class="colab-df-container">
    <div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>MONTH</th>
      <th>flood_months</th>
      <th>total_months</th>
      <th>flood_pct</th>
      <th>avg_precip_in</th>
      <th>avg_soil_moisture</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1.0</td>
      <td>10</td>
      <td>10.0</td>
      <td>2.99</td>
      <td>0.463</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>2.0</td>
      <td>10</td>
      <td>20.0</td>
      <td>2.85</td>
      <td>0.467</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>0.0</td>
      <td>10</td>
      <td>0.0</td>
      <td>2.48</td>
      <td>0.443</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>2.0</td>
      <td>10</td>
      <td>20.0</td>
      <td>3.54</td>
      <td>0.417</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>5.0</td>
      <td>10</td>
      <td>50.0</td>
      <td>4.52</td>
      <td>0.407</td>
    </tr>
    <tr>
      <th>5</th>
      <td>6</td>
      <td>3.0</td>
      <td>10</td>
      <td>30.0</td>
      <td>3.76</td>
      <td>0.377</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>8.0</td>
      <td>10</td>
      <td>80.0</td>
      <td>3.87</td>
      <td>0.358</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8</td>
      <td>6.0</td>
      <td>10</td>
      <td>60.0</td>
      <td>3.94</td>
      <td>0.380</td>
    </tr>
    <tr>
      <th>8</th>
      <td>9</td>
      <td>4.0</td>
      <td>10</td>
      <td>40.0</td>
      <td>3.79</td>
      <td>0.379</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>3.0</td>
      <td>10</td>
      <td>30.0</td>
      <td>3.36</td>
      <td>0.413</td>
    </tr>
    <tr>
      <th>10</th>
      <td>11</td>
      <td>1.0</td>
      <td>10</td>
      <td>10.0</td>
      <td>2.91</td>
      <td>0.423</td>
    </tr>
    <tr>
      <th>11</th>
      <td>12</td>
      <td>3.0</td>
      <td>10</td>
      <td>30.0</td>
      <td>3.17</td>
      <td>0.440</td>
    </tr>
  </tbody>
</table>

```python
#query 3: compare conditions in flood vs non-flood months
q3 = con.execute("""
    SELECT
        HAD_FLOOD,
        COUNT(*)                                AS n_months,
        ROUND(AVG(PRCP_TOTAL), 2)               AS avg_precip,
        ROUND(AVG(PRCP_MAX), 2)                 AS avg_max_daily_rain,
        ROUND(AVG(HEAVY_RAIN_DAYS), 2)          AS avg_heavy_rain_days,
        ROUND(AVG(SOIL_MOISTURE_MEAN), 3)       AS avg_soil_moisture,
        ROUND(AVG(DROUGHT_INDEX), 3)            AS avg_drought_index
    FROM master
    GROUP BY HAD_FLOOD
    ORDER BY HAD_FLOOD
""").df()
q3['HAD_FLOOD'] = q3['HAD_FLOOD'].map({0: 'No flood', 1: 'Flood'})
print('Average conditions — flood months vs non-flood months:')
display(q3)
```

    Average conditions — flood months vs non-flood months:




  <div id="df-e86fb242-a35f-4667-aadd-c73037af8563" class="colab-df-container">
    <div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>HAD_FLOOD</th>
      <th>n_months</th>
      <th>avg_precip</th>
      <th>avg_max_daily_rain</th>
      <th>avg_heavy_rain_days</th>
      <th>avg_soil_moisture</th>
      <th>avg_drought_index</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>No flood</td>
      <td>82</td>
      <td>2.62</td>
      <td>0.96</td>
      <td>0.55</td>
      <td>0.417</td>
      <td>0.525</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Flood</td>
      <td>38</td>
      <td>5.18</td>
      <td>1.81</td>
      <td>1.58</td>
      <td>0.407</td>
      <td>0.300</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-e86fb242-a35f-4667-aadd-c73037af8563')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>


## Model Implementation


```python
import pandas as pd
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import StratifiedKFold, cross_val_predict
from sklearn.metrics import classification_report, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

#loading master table and defining features
master = pd.read_parquet('data/master.parquet')

FEATURES = [
    'PRCP_TOTAL_LAG1',
    'PRCP_MAX_LAG1',
    'HEAVY_RAIN_DAYS',
    'TMAX_MEAN',
    'SOIL_MOISTURE_MEAN_LAG1',
    'SOIL_MOISTURE_MAX_LAG1',
    'DROUGHT_INDEX_LAG1',
    'MONTH'
]
TARGET = 'HAD_FLOOD'

#drop the first row (NaN from lag) and any other nulls
model_df = master[FEATURES + [TARGET]].dropna()
X = model_df[FEATURES]
y = model_df[TARGET]

print(f'Samples: {len(model_df)}')
```

    Samples: 119



```python
#train model with cross-validation
rf = RandomForestClassifier(
    n_estimators=100,
    max_depth=4,             #shallow to avoid overfitting on small dataset
    class_weight='balanced', #corrects for more flood months than non-flood
    random_state=88
)

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=88)

#cross_val_predict gives one prediction per sample across all folds
y_pred = cross_val_predict(rf, X, y, cv=cv)

print(classification_report(y, y_pred, target_names=['No flood', 'Flood']))

#fit on full dataset so we can extract feature importances
rf.fit(X, y)
```

                  precision    recall  f1-score   support
    
        No flood       0.81      0.85      0.83        81
           Flood       0.65      0.58      0.61        38
    
        accuracy                           0.76       119
       macro avg       0.73      0.72      0.72       119
    weighted avg       0.76      0.76      0.76       119
    
<div id="sk-container-id-2" class="sk-top-container"><div class="sk-text-repr-fallback"><pre>RandomForestClassifier(class_weight=&#x27;balanced&#x27;, max_depth=4, random_state=88)</pre><b>In a Jupyter environment, please rerun this cell to show the HTML representation or trust the notebook. <br />On GitHub, the HTML representation is unable to render, please try loading this page with nbviewer.org.</b></div><div class="sk-container" hidden><div class="sk-item"><div class="sk-estimator fitted sk-toggleable"><input class="sk-toggleable__control sk-hidden--visually" id="sk-estimator-id-2" type="checkbox" checked><label for="sk-estimator-id-2" class="sk-toggleable__label fitted sk-toggleable__label-arrow"><div><div>RandomForestClassifier</div></div><div><a class="sk-estimator-doc-link fitted" rel="noreferrer" target="_blank" href="https://scikit-learn.org/1.6/modules/generated/sklearn.ensemble.RandomForestClassifier.html">?<span>Documentation for RandomForestClassifier</span></a><span class="sk-estimator-doc-link fitted">i<span>Fitted</span></span></div></label><div class="sk-toggleable__content fitted"><pre>RandomForestClassifier(class_weight=&#x27;balanced&#x27;, max_depth=4, random_state=88)</pre></div> </div></div></div></div>




```python
#predicted flood probability by condition
#uses the fitted random forest to show how flood probability changes across low / medium / high values of the three strongest predictors

rf.fit(X, y)
baseline = model_df.median()[FEATURES]  #start from median conditions

#potential scenarios for flood probability
scenarios = {
    'Baseline (median conditions)':          {},
    'Low prior precipitation (<1 in)':       {'PRCP_TOTAL_LAG1': 0.8},
    'High prior precipitation (>4 in)':      {'PRCP_TOTAL_LAG1': 4.5},
    'Low soil moisture (<0.22)':             {'SOIL_MOISTURE_MEAN_LAG1': 0.20},
    'High soil moisture (>0.35)':            {'SOIL_MOISTURE_MEAN_LAG1': 0.37},
    'No drought (index = 0)':               {'DROUGHT_INDEX_LAG1': 0.0},
    'Moderate drought (index > 1)':          {'DROUGHT_INDEX_LAG1': 1.5},
    'High precip + high soil moisture':      {'PRCP_TOTAL_LAG1': 4.5,
                                              'SOIL_MOISTURE_MEAN_LAG1': 0.37},
}

#takes median value and updates probability based on flood occurrance
rows = []
for label, overrides in scenarios.items():
    scenario = baseline.copy()
    for col, val in overrides.items():
        scenario[col] = val
    prob = rf.predict_proba(scenario.values.reshape(1, -1))[0][1]
    rows.append({'Scenario': label, 'Flood probability': f'{prob*100:.1f}%'})

scenario_df = pd.DataFrame(rows)
print(f'Overall flood rate in data: {y.mean()*100:.1f}%\n')
display(scenario_df)
```

    Overall flood rate in data: 31.9%
    




  <div id="df-38f3d92b-fa8a-4b22-a0fc-7d26e38594ce" class="colab-df-container">
    <div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Scenario</th>
      <th>Flood probability</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Baseline (median conditions)</td>
      <td>57.0%</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Low prior precipitation (&lt;1 in)</td>
      <td>42.4%</td>
    </tr>
    <tr>
      <th>2</th>
      <td>High prior precipitation (&gt;4 in)</td>
      <td>62.8%</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Low soil moisture (&lt;0.22)</td>
      <td>53.5%</td>
    </tr>
    <tr>
      <th>4</th>
      <td>High soil moisture (&gt;0.35)</td>
      <td>54.3%</td>
    </tr>
    <tr>
      <th>5</th>
      <td>No drought (index = 0)</td>
      <td>57.0%</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Moderate drought (index &gt; 1)</td>
      <td>55.0%</td>
    </tr>
    <tr>
      <th>7</th>
      <td>High precip + high soil moisture</td>
      <td>58.9%</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-38f3d92b-fa8a-4b22-a0fc-7d26e38594ce')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>


## Analysis Rationale

A random forest classifier was selected for this analysis because it handles the small dataset size (120 monthly observations), mixed feature types, and class imbalance between flood and non-flood months well. Shallow trees with a maximum depth of four were used to prevent overfitting on a dataset of this size, and the class_weight='balanced' parameter was set to correct for the fact that flood months outnumber non-flood months roughly two to one in Albemarle County over the study period. Features were constructed as lag-1 values of the prior month's precipitation, soil moisture, and drought conditions rather than same-month values in order to preserve the causal direction of the model. The goal is to predict whether a flood will occur based on antecedent environmental conditions, not to describe conditions during a flood that has already happened. Five-fold stratified cross-validation was used to evaluate model performance, which ensures that the flood-to-non-flood ratio is preserved across all folds and that every month in the dataset is used for both training and testing exactly once, giving an honest estimate of how the model would perform on unseen data. The primary evaluation metric is the classification report rather than raw accuracy, since accuracy alone is misleading when one class is more common than the other (for example, a model that always predicts "flood" would achieve over 60% accuracy in this dataset without learning anything useful).

## Visualizations


```python
#visualization 1: feature importance
importances = pd.Series(rf.feature_importances_, index=FEATURES).sort_values()

fig, ax = plt.subplots(figsize=(7, 4))
importances.plot(kind='barh', ax=ax, color='lightblue')
ax.set_xlabel('Importance')
ax.set_title('Which conditions best predict Albemarle County flooding?',
             fontweight='bold')
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
plt.tight_layout()
plt.savefig('data/feature_importance.png', dpi=150, bbox_inches='tight')
plt.show()
```


    
![png](file_2_pipeline_files/file_2_pipeline_14_0.png)
    



```python
#visualization 2: flood events and key predictors over time
plot_df = master.dropna(subset=['SOIL_MOISTURE_MEAN', 'PRCP_TOTAL']).copy()
plot_df['DATE'] = pd.to_datetime(plot_df[['YEAR','MONTH']].assign(DAY=1))
flood_dates = plot_df[plot_df['HAD_FLOOD'] == 1]['DATE']

fig, axes = plt.subplots(3, 1, figsize=(12, 7), sharex=True)
fig.suptitle('Albemarle County Flood Predictors 2015–2024',
             fontweight='bold', fontsize=12)

#precipitation
axes[0].bar(plot_df['DATE'], plot_df['PRCP_TOTAL'],
            color='steelblue', alpha=0.7, width=25)
axes[0].set_ylabel('Precip (in)')

#soil moisture
axes[1].plot(plot_df['DATE'], plot_df['SOIL_MOISTURE_MEAN'],
             color='saddlebrown', linewidth=1.5)
axes[1].set_ylabel('Soil moisture\n(m³/m³)')

#drought index
axes[2].plot(plot_df['DATE'], plot_df['DROUGHT_INDEX'],
             color='darkorange', linewidth=1.5)
axes[2].set_ylabel('Drought index\n(0–5)')

#mark flood months on all panels
for ax in axes:
    for d in flood_dates:
        ax.axvline(d, color='black', alpha=0.25, linewidth=1)
    ax.spines['top'].set_visible(False)
    ax.spines['right'].set_visible(False)

#single shared legend
axes[0].axvline(flood_dates.iloc[0], color='black', alpha=0.6,
                linewidth=1, label='Flood month')
axes[0].legend(fontsize=9, loc='upper right')

plt.tight_layout()
plt.savefig('data/flood_predictors_timeseries.png', dpi=150, bbox_inches='tight')
plt.show()
```


    
![png](file_2_pipeline_files/file_2_pipeline_15_0.png)
    



```python
#visualization 3: confusion matrix
fig, ax = plt.subplots(figsize=(4, 3.5))
ConfusionMatrixDisplay.from_predictions(
    y, y_pred,
    display_labels=['No flood', 'Flood'],
    cmap='Blues',
    colorbar=False,
    ax=ax
)
ax.set_title('Flood prediction, 5-fold cross-validation',
             fontweight='bold', fontsize=10)
plt.tight_layout()
plt.savefig('data/confusion_matrix.png', dpi=150, bbox_inches='tight')
plt.show()
```


    
![png](file_2_pipeline_files/file_2_pipeline_16_0.png)
    


# Visualization Rationale


**Visualization 1:**

The feature importance chart was chosen because it directly helps answer the question of the project, which is which environmental conditions actually drive flood risk in Albemarle County. The visualization puts this information in a form that is interpretable to a non-technical audience. A horizontal bar chart was used rather than a table because the visual ranking makes it immediately clear which predictors dominate without requiring the reader to scan numbers. The chart is fitted on the full dataset rather than a single cross-validation fold so the importances reflect the model's overall learned behavior rather than one split of the data.

**Visualization 2:**

The three-panel time series was chosen to show the raw temporal patterns in the data before any modeling, which frames the results in observable reality rather than just model outputs. Sharing the x-axis across all three panels lets the reader visually see how precipitation, soil moisture, and drought conditions have moved together over time and compare those movements against the gray flood month markers. This graph is particularly useful for communicating the lag relationship because you can see that flood months often follow periods of elevated soil moisture and precipitation rather than coinciding with drought spikes. This helps validate the modeling decision to use lag-1 features.

**Visualization 3:**

The confusion matrix was chosen because ROC-AUC and accuracy scores alone are abstract and hard to communicate to a non-technical audience. It makes the two types of mistakes the model can make easy to see, which are predicting a flood that doesn't happen (false positive) and missing a flood that does (false negative). It allows a reader to judge which failure mode is more acceptable for their use case. Cross-validation predictions were used rather than training-set predictions to ensure the matrix reflects out-of-sample performance rather than overfitted results on data the model has already seen.
