# Energy Consumption Prediction for Ro-Ro Passenger Ships using ML

This project focuses on predicting the fuel-based energy consumption of the Danish Ro-Ro passenger ship *MS Smyril* using real-world operational data. By applying machine learning, we aim to support sustainable maritime logistics through intelligent fuel usage optimization.

## 🚢 Project Objective

Fuel efficiency is essential for shipping companies due to environmental regulations and cost pressures. This project develops regression models to predict energy consumption from variables such as ship speed, rudder angles, and environmental conditions. Accurate predictions can support smarter operational decisions.

## 🗃️ Data Handling & Preprocessing

All processing was performed in **Google Colab** to ensure reproducibility and scalability.

### 🔗 Setup
- Raw datasets were stored in a folder named `ML-MP1` in Google Drive.
- Google Drive was mounted in Google Colab.
- At least 2 GB of storage was required.

### ⏱️ Timestamp Alignment
- Raw timestamps were in .NET Epoch format.
- Converted to Unix Epoch (in seconds) for consistency.
- Used `merge_asof()` (instead of a regular `merge()`) to align asynchronously sampled data across files.
- Applied a **1-second tolerance** for near-perfect alignment without data loss.

### 🧹 Data Cleaning
- All rows with any missing (NaN) values were removed.
- This reduced the dataset from 1.6 million to ~980,000 rows.

### ⚡ Energy Consumption Calculation
A new `energy_consumption` column was computed per row using:

```python
df["energy_consumption"] = (fuelDensity × fuelVolumeFlowRate × 3600 × 24) / 1000
```

This gave standardized kWh/day values suitable for modeling.

### 🗺️ Geolocation Conversion
- Latitude and longitude were originally in DDMM.MMMM format.
- Converted to decimal degrees (DD.DDDDDD) for mapping and modeling.

### ⏳ Time Aggregation
- Timestamps were irregularly spaced.
- Aggregated to **1-minute intervals** using `resample()`:
  - `mean()` for sensor-based values (e.g., wind speed)
  - `sum()` for cumulative values (e.g., fuel volume flow rate)

### 📌 Feature Selection
- Features with very high correlation (e.g., 1.0) were removed to avoid redundancy.
- Features with very low correlation (<0.1) were excluded.
- Removed multicollinear variables (e.g., those strongly correlated with already selected ones).

#### Final Selected Features
- `speedThroughWater`
- `portRudderAngle`
- `starboardLevelMeasurements`
- `longitude`
- `starboardRudderAngle`
- `windSpeed`
- `fuelTemperature`
- `portLevelMeasurements`
- `fuelDensity`
- `starboardPropellerPitch`

## 🧠 Modeling Approach

### 1. 🌳 Random Forest Regressor
- Used 100 estimators
- Trained on 80% of data
- Tested on 20%
- No feature scaling needed
- Provided feature importance

#### Performance
- Initial R² ≈ 0.835
- Improved to **0.8984** after including `starboardPropellerPitch`

### 2. 🚀 XGBoost Regressor
- Used 200 estimators
- Learning rate: 0.05
- Max depth: 6
- Trained and tested using the same 80/20 split

#### Performance
- R² ≈ **0.9033** from the start (better than Random Forest)

### 📊 Evaluation Metrics
- Mean Absolute Error (MAE)
- Root Mean Squared Error (RMSE)
- R² Score
- Scatter plots for visualizing predicted vs actual consumption

## ⚠️ Key Challenges & Solutions

### 1. High Memory Usage on Merging Large CSVs
- Standard join operations caused crashes.
- ✅ **Solution**: Switched to `merge_asof()` and processed row-by-row to conserve memory.

### 2. Timestamp Misalignment Between Files
- Different files sampled at different rates.
- ✅ **Solution**: Used `merge_asof()` with a 1-second tolerance.

### 3. Redundant or Irrelevant Features
- Some features had zero or overly strong correlation.
- ✅ **Solution**: Used correlation heatmaps and domain logic to select meaningful features.

### 4. ML vs Physics Modeling
- Fuel consumption can also be estimated via physical equations (speed, resistance).
- ✅ **Solution**: ML chosen for its ability to learn **nonlinear** and **complex feature interactions**.

## 📈 Results Summary

| Model            | R² Score | Notes                                  |
|------------------|----------|----------------------------------------|
| Random Forest    | 0.8984   | Improved after adding key features     |
| XGBoost          | 0.9033   | Best performance with default settings |

## 📁 Output Files
- `merged_data_aggregated.csv`: Final preprocessed dataset
- Scatter plots: Actual vs predicted energy consumption

## 🔮 Future Improvements
- Hyperparameter tuning (grid/random search)
- Cross-validation for stability
- Benchmarking with LightGBM or CatBoost
- Integration of physics-based hybrid models

## 👨‍💻 Author

**Moh’d Abu Quttain**  
