# Smart Data Pipeline for Road Traffic Management in Denmark

> *Objective: Predict real-time road trip durations and suggest alternative routes to avoid congestion — using Spark, Delta Lake, and Random Forests.*

This Jupyter notebook implements an **intelligent data pipeline** for road traffic management in Denmark, based on the **CityPulse** dataset. It combines:
- **Data cleaning and preparation** with **PySpark**
- **Predictive modeling** with **Random Forest Regressor**
- **Optimized storage** with **Delta Lake**
- **Visualization and decision support** (congestion alerts, alternative route suggestions)
- **Geospatial computation** with `geopy` to propose realistic alternatives

---

## Context & Objective

Road traffic in Denmark is monitored via sensors and reports (CityPulse). The goal of this project is to:

1. **Predict trip duration** between two points based on:
   - Average speed (`NDT_IN_KMH`)
   - Distance (`DISTANCE_IN_METERS`)
   - Geographic coordinates (latitude/longitude of origin and destination)

2. **Detect trips at risk of congestion** (predicted duration > threshold)

3. **Suggest uncongested alternative routes**, ensuring they are:
   - Sufficiently long (> 2.5 km)
   - Different from congested routes
   - Not loops (origin ≠ destination)

---

## Target Audience

| Audience | What They Will Find |
|----------|----------------------|
| **Traffic Managers / Smart Cities** | An operational prototype to anticipate congestion and propose detours. |
| **Students in Data Engineering / ML** | A complete example of a Spark + ML + Delta Lake + geospatial pipeline. |
| **Data Engineers / Data Scientists** | A clean, modular implementation with best practices (caching, cross-validation, Delta storage). |
| **Decision Makers / Curious Readers** | A concrete demonstration of how data can improve urban mobility. |

---

## Technical Steps Implemented

### 1. Data Loading & Cleaning
- Read `CityPulseTrafic.csv` with schema inference
- Remove null values (`na.drop()`)
- Convert GPS coordinates to `double` type

### 2. Feature Engineering for ML
- Assemble features using `VectorAssembler`:
  - `NDT_IN_KMH`, `DISTANCE_IN_METERS`, `POINT_1_LAT/LNG`, `POINT_2_LAT/LNG`
- Normalize using `MinMaxScaler` → scale to [0,1]

### 3. Storage in Delta Lake
- Write cleaned data to `/mnt/delta/BDTraffic_cleaned`
- Benefits: ACID transactions, versioning, performance, Spark compatibility

### 4. Modeling with Random Forest
- Split data into train/test (80/20)
- Cross-validation over 2 folds with hyperparameter grid:
  - `numTrees`: [50, 100]
  - `maxDepth`: [10, 20]
  - `maxBins`: [64, 128]
- Evaluation metric: **RMSE** (Root Mean Squared Error)

> ✅ **Final RMSE: 17.73 seconds** → high accuracy for trip duration predictions.

### 5. Visualization & Analysis
- Plot comparing actual vs. predicted durations
- Descriptive statistics of predictions
- Calculation of the **95th percentile** → used to set alert threshold

### 6. Congestion Detection
- Threshold set at **137 seconds** (~95th percentile)
- Add `alert` column = `True` if prediction > threshold
- Display at-risk trips:
  - e.g., `Landevejen → Landevejen` (234.78 sec), `Randersvej → Vejlby Centervej` (200.42 sec)

### 7. Alternative Route Suggestions
- Use `geopy.distance.geodesic` to compute real distances between points
- Filter routes by:
  - Not congested
  - Distance > 2.5 km
  - No loops (origin street ≠ destination street)
  - No duplicates
- Example suggestions:
  - `Hovedvejen → Møllebakken` (3.33 km)
  - `Århusvej → Oddervej` (12.84 km)
  - `Nordjyske Motorvej → 15` (2.65 km)

---

## Technologies & Libraries

```python
from pyspark.sql import SparkSession
from pyspark.ml.feature import VectorAssembler, MinMaxScaler
from pyspark.ml.regression import RandomForestRegressor
from pyspark.ml.evaluation import RegressionEvaluator
from pyspark.ml.tuning import ParamGridBuilder, CrossValidator
from delta import *
from geopy.distance import geodesic
import matplotlib.pyplot as plt
