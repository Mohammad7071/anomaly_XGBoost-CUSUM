# Oil Production Anomaly Detection Using XGBoost and CUSUM

## Overview

This project combines Machine Learning and Statistical Process Monitoring to identify unusual changes in oil production.

An XGBoost regression model is used to estimate expected oil production, while the Cumulative Sum (CUSUM) method analyzes prediction errors to detect persistent deviations from expected production.

The current analysis focuses on a single oil well, **Well A01**, using daily production data.

## Project Objectives

The main objective is to develop a data-driven approach for monitoring oil production and identifying potential anomalies.

The project aims to estimate expected oil production, analyze the differences between actual and predicted values, detect persistent positive and negative deviations, and highlight unusual production patterns for further investigation.

## Dataset

The analysis uses daily production data with the following variables:

| Variable | Description |
|---|---|
| `well_id` | Well identifier |
| `date` | Observation date |
| `actual_oil_bopd` | Actual oil production rate (barrels of oil per day) |
| `pressure_psi` | Pressure measurement |
| `water_cut_pct` | Water cut percentage |

The current implementation focuses on Well A01.

## Methodology

### 1. Data Preparation

The dataset is filtered to select Well A01 and sorted chronologically. Missing values in the selected features are handled before model training.

### 2. Feature Engineering

Lagged features are created using previous observations:

- Previous oil production (`oil_lag_1`)
- Previous pressure (`pressure_lag_1`)
- Previous water cut (`water_cut_lag_1`)

These variables are used as inputs to the XGBoost regression model.

### 3. Oil Production Prediction Using XGBoost

XGBoost is applied to estimate expected oil production.

A walk-forward approach is used, where the model is trained on historical observations before predicting the next observation.

The first 365 observations are used to initialize the forecasting process. The model is then retrained at each prediction step using the available historical data.

### 4. Prediction Error Calculation

The prediction error is calculated as:

`Error = Actual Oil Production - Predicted Oil Production`

Positive errors indicate that actual production is higher than predicted production, while negative errors indicate lower-than-predicted production.

### 5. Calibration and Testing

The out-of-sample prediction results are divided into two periods:

- **Calibration:** First 180 prediction errors.
- **Testing:** Remaining prediction errors.

The calibration period is used to calculate the baseline mean and standard deviation of prediction errors.

### 6. CUSUM Anomaly Detection

The Cumulative Sum (CUSUM) method is applied to standardized prediction errors.

The standardized error is calculated as:

`z(t) = (Error(t) - Baseline Mean) / Baseline Standard Deviation`

Two cumulative statistics are calculated:

**Positive CUSUM:**

`C+(t) = max(0, C+(t-1) + z(t) - k)`

**Negative CUSUM:**

`C-(t) = max(0, C-(t-1) - z(t) - k)`

The parameters used in the initial implementation are:

- `k = 0.5` — Reference value.
- `h = 5` — Decision threshold.

Both CUSUM statistics start at zero at the beginning of the testing period.

### 7. Anomaly Interpretation

Potential anomalies are identified when either CUSUM statistic exceeds the decision threshold.

- **Overproduction:** Positive CUSUM exceeds the threshold, indicating persistent positive prediction errors.
- **Underproduction:** Negative CUSUM exceeds the threshold, indicating persistent negative prediction errors.
- **Normal:** Neither statistic exceeds the threshold.

These signals indicate unusual deviations and should be investigated further. They do not independently confirm the underlying operational or reservoir-related causes.

## Technologies Used

- Python
- Pandas
- NumPy
- XGBoost
- Jupyter Notebook

## How to Run

1. Clone this repository:

   `git clone https://github.com/Mohammad7071/anomaly_XGBoost-CUSUM.git`

2. Navigate to the project directory:

   `cd anomaly_XGBoost-CUSUM`

3. Install the required libraries:

   `pip install pandas numpy xgboost matplotlib jupyter`

4. Open the Jupyter Notebook:

   `jupyter notebook`

5. Open `residual_analysis+CUSUM.ipynb` and run the cells.

**Note:** The dataset must be available separately. Make sure it contains the required columns described in the Dataset section and update the data-loading path if necessary.

## Project Limitations

- The current analysis focuses on one well (A01).
- The lagged features use the previous available observation, which may not always represent the previous calendar day.
- The model is retrained at every prediction step, which can increase computational time.
- The initial CUSUM parameters are illustrative and have not necessarily been optimized for the dataset.
- CUSUM performance depends on the statistical properties of calibration errors.
- Autocorrelation and changes in the baseline error distribution may affect anomaly detection.
- Detected signals do not establish the actual cause of production changes.

## Future Improvements

- Extend the analysis to multiple wells.
- Optimize CUSUM parameters based on the desired false alarm rate.
- Evaluate model performance using forecasting metrics such as MAE and RMSE.
- Investigate detected anomalies using additional operational and reservoir data.
- Compare CUSUM with other statistical anomaly detection techniques.
- Evaluate the stability and autocorrelation of prediction errors.

## Conclusion

This project demonstrates how Machine Learning and statistical monitoring can be combined for oil production anomaly detection.

XGBoost estimates expected production, while CUSUM identifies persistent deviations from model predictions. This approach can support production monitoring by highlighting unusual patterns that require further investigation.

## Disclaimer

This project is an educational and analytical study. The detected anomalies are statistical signals and should not be interpreted as confirmed operational failures or reservoir problems.
