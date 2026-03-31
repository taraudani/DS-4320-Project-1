# The Flood Risk is Written in the Soil - How Albemarle County Environmental Data Predicts Flooding

## Hook
Every time Albemarle County floods, the warning signs have already been there, not in the sky, but in the ground beneath it. 
Soil saturated from weeks of rain, a watershed primed by antecedent moisture, and fast breaking droughts. 
These are the conditions that turn an ordinary storm into a flood emergency. 
Yet most local flood warnings arrive hours before an event, leaving residents and governments with little time to prepare. 
This new data analysis of a decade of environmental monitoring records shows that the conditions predicting Albemarle County flooding are measurable weeks in advance, and that the data to do so has been publicly available all along.

## Problem Statement
Between 2015 and 2025, Albemarle County recorded 143 flood and flash flood events, which is more than any other category of extreme weather in the area during that period. 2018 and 2020 were the most severe years, with 41 and 31 events respectively, causing significant property damage and disruption to residents and local infrastructure. 
Despite this consistent pattern of flooding, most early warning systems operate on short time horizons, issuing alerts only once a storm is already underway. 
What is missing is a longer-horizon risk alert, something that tells emergency managers and local planners not just that a flood is happening now, but that one is likely to happen in the coming weeks based on current environmental conditions so that people can prepare. 
The challenge is not a lack of data. NOAA, NASA, the US Drought Monitor, and the Open-Meteo climate archive all provide free and publicly accessible environmental monitoring records for Albemarle County that goes back decades.
The challenge is that these data sources have never been brought together into a single predictive framework that is oriented specifically toward flood risk in this watershed.

## Solution Description
This project assembles these four publicly available environmental data sources: NOAA storm event records, daily climate observations from the Charlottesville Albemarle Airport, ERA5 soil moisture reanalysis data from the Open-Meteo API, and weekly US Drought Monitor classifications for Albemarle County. 
They are assembled into a unified monthly dataset spanning 2015 through 2025. A random forest classification model is trained on this dataset to predict whether a flood event will occur in a given month based on the prior month's environmental conditions. 
By using prior-month values of precipitation, soil moisture, and drought status as predictors, the model is designed to identify flood risk before a storm arrives rather than during one. 
The result is a practical, data-driven tool that could support longer-horizon flood preparedness decisions at the county level. 
Every data source used is free, publicly accessible, and updated on a regular cadence, which is crucial to reproducibility and could be extended to future years or adapted by any similarly monitored community.

## Chart

<img width="1211" height="693" alt="Screenshot 2026-03-31 at 1 20 41 PM" src="https://github.com/user-attachments/assets/fad35972-a1c1-4ab7-8c39-f8a55f4ee3bd" />


The chart above shows monthly precipitation, soil moisture, and drought conditions in Albemarle County from 2015 through 2025, with gray vertical lines marking months in which at least one flood event was recorded. 
Several patterns are immediately visible, as flood months cluster heavily in periods of elevated soil moisture, confirming that antecedent ground saturation is a stronger predictor of flooding than any single storm event in isolation. 
The drought index drops sharply in many of the months immediately preceding flood clusters, capturing the transition from dry to wet conditions that is particularly dangerous for a watershed that has lost its absorption capacity. 
These three signals together form the core of a predictive framework that is grounded in the physical mechanisms of flooding rather than statistical pattern-matching alone.
