# Chicago Crimes Time-Series Forecasting
Author: Hiba Shanti

Introduction: 

The Chicago Crimes Dataset logs reported crimes in Chicago from 2001 to the present. Sourced from the Chicago Police Department, it contains over 7.9 million records. Each entry includes the crime type, date, location type, and whether an arrest was made. This consistent timeline makes it ideal for studying seasonal trends and forecasting future crime rates.

## Data Cleaning & Preprocessing

The raw dataset required extensive cleaning to fix overlapping files, handle missing data, and prepare a clean timeline for modeling.

### 1. Removing Duplicates
*   **The Issue:** Merging the annual CSV files created **1,948,389 duplicate records**.
*   **The Fix:** Removed identical rows so that each real-world crime is only counted once. This prevents artificial inflation of crime frequencies.

  ### 2. Handling Missing Values (NaN)
*   **The `Ward` Column:** Contained over **600,000 missing values**. Deleting these rows would create massive gaps in our timeline. Because ward numbers are not needed for city-wide forecasting, the entire column was dropped.
*   **Geographic & Location Data:** Dropped rows with missing `Latitude`, `Longitude`, or `District` entries to keep spatial data accurate. Filled missing `Location Description` fields with `'UNKNOWN'`.

*   ## Key Insights for Media Reporting

This section answers specific questions regarding historical crime trends in Chicago to provide data-driven insights for local news coverage.

### Topic 2: Long-Term Annual Trends
*   **Overall Trend:** Between 2001 and 2022, the total volume of reported crime in Chicago has been **decreasing** overall.

*   <img width="597" height="449" alt="image" src="https://github.com/user-attachments/assets/b07c6974-5152-44c1-abe3-6c262049c0b6" />

*  **Trend Anomalies:** The garph confirms that the categories are following a consistent long-term downward trend from 2001 to 2022. However, a key short-term anomaly occurs at the tail end of the dataset (2021–2022), where **Theft and Criminal Trespass** experienced a sharp upward rebound, defying the ongoing decline seen in other violent categories.

* <img width="326" height="227" alt="Screen Shot 2026-05-13 at 5 13 43 PM" src="https://github.com/user-attachments/assets/88af1ea1-03c9-48ca-ab67-70ffea3f7c70" />



### Topic 3: AM vs. PM Rush Hour Analysis
*   **Volume Comparison:** Criminal incidents are more common during **PM** rush hour [4:00 PM - 7:00 PM]).
*   **Top 5 AM Rush Hour Crimes:**
    1. THEFT
    2. BATTERY
    3. CRIMINAL DAMAGE
    4. BURGLARY
    5. OTHER OFFENSE
*   **Top 5 PM Rush Hour Crimes:**
    1. THEFT
    2. BATTERY
    3. CRIMINAL DAMAGE	
    4. NARCOTICS
    5. ASSAULT

### Topic 4: Monthly Seasonality Patters
*   **Peak & Trough Months:** City-wide crime volume traditionally peaks during the month of **July** and hits its lowest levels during **February**.
* In the monthly trend graph, January appears to be the peak,In fact,Its only  representing the most abrupt upward structural shift relative to its surrounding months, not because it has more total crimes than July.

* ### Topic 5: Holiday Crime Analysis

This section explores how major holidays impact crime volumes and profiles in Chicago, providing insights into which days experience the highest activity.

*   **Top 3 Holidays with the Most Crime:**
    1. **New Year's Day**
    2. **Independence Day**
    3. **Labor Day**

#### Top 5 Most Common Crimes by Top Holidays:

*   **On New Year's Day:**
    1. THEFT                
    2. BATTERY             
    3. CRIMINAL DAMAGE       
    4. OTHER OFFENSE         
    5. DECEPTIVE PRACTICE
    
*   **On Independence Day:**
    1. BATTERY            
    2. THEFT              
    3. CRIMINAL DAMAGE    
    4. ASSAULT           
    5. NARCOTICS

*   **On Labor Day:**
    1. BATTERY           
    2. THEFT
    3. CRIMINAL DAMAGE    
    4. NARCOTICS         
    5. ASSAULT
 

Part 2 
    ## SARIMA Time-Series Modeling & Forecasting

This section documents the structured workflow used to build, evaluate, tune, and deploy the time-series forecasting models for 2 types of Crimes: **Theft and Battery**.

### Starting with The Theft Crime: 
### 1. Data Transformation & Preparation
The raw transactional crime logs were aggregated into an unbroken, uniform monthly sequence:
*   **Aggregation:** Captured monthly volumes using `.resample('MS').size()
  
### 2. Time-Series Decomposition & Stationarity
*   **Seasonal Decomposition:** A seasonal decomposition was applied to the monthly crime data to break it down into three clean components: **Trend**, **Seasonal Component**, and **Residuals**. The seasonal graph isolated a powerful, recurring 12-month rhythm that repeats perfectly every year.
*   **Stationarity Check:** The raw monthly timeline was statistically tested for stationarity. The initial test returned a **false result (non-stationary)**, meaning the average crime numbers and variance were changing over time instead of remaining flat. 
*   **Differencing Strategy:** To fix this non-stationarity and make the data ready for modeling, two types of transformations were applied:
    *   **Non-Seasonal Differencing ($d=1$):** 
    *   **Seasonal Differencing ($D=1, s=12$):** 


### 3. Model Identification (ACF & PACF)
Autocorrelation (ACF) and Partial Autocorrelation (PACF) plots were generated on the fully differenced, stationary data:
**   **Initial Estimates:** The sharp spikes at the beginning of the PACF plot helped estimate the Autoregressive orders ($p$, $P$). The gradually dying out pattern (geometric decay) in the ACF plot guided the initial Moving Average orders ($q$, $Q$).
Therfore the SARIMA model was used to predict the results

<img width="989" height="490" alt="image" src="https://github.com/user-attachments/assets/fdc9a525-df88-49a6-80d4-c1c6de9e779e" />


### 4. Evaluation Strategy (Train-Test Split)
*   **Training Set:** All historical data up to **June 2022**.
*   **Testing Set (Validation):** The final **6 months of 2022** (July 2022 through December 2022). This holdout window was used to rigorously backtest baseline predictive power.

### 5. Manual Model Fit vs. Automated Hyperparameter Tuning
Two distinct modeling iterations were conducted to discover the optimal parameters:

1.  **Manual SARIMA:** Constructed manually using the $(p,d,q) \times (P,D,Q)_{12}$ combinations identified during exploratory analysis.
2.  **Automated Tuning (`pmdarima.auto_arima`):** Executed a comprehensive grid search targeting the lowest **(AIC)** score to dynamically balance model complexity and overfitting risks.

### 6. Model Evaluation Metrics
Both configurations were fitted on the training partition and evaluated against the true 6-month holdout test set using standard error metrics.

### 7. Final Model Selection & Justification
The **[Auto-ARIMA]** model for the Theft crime data was selected as the final production engine. This selection is  justified by:

  - The MAPE result is good. It means that the predictions are off by only 6% on average. 

  - Also, in the correlgram plot, most of the points are within the shaded area, this indicates that the model has successfully captured most of the patterns in the data.

  - However, when comparing the testing and forecasting data, the forecast is slightly "stiff." It captures the downward slope but doesn't quite match the sharpness of the dips in the test data. Therefore, we might tune the model to make some improvenmnts.

 * The forecast for the tested data
    <img width="839" height="393" alt="image" src="https://github.com/user-attachments/assets/5ee15bbd-08f6-41ff-a5b3-9a56813178b0" />

* The diagnostcs for the tested data
<img width="593" height="455" alt="image" src="https://github.com/user-attachments/assets/8cc75ba5-2aa5-45ce-b4fa-d870c7dac7d2" />

 
### 8. Production Deploy & True Future Forecasts
After fixing the training boundaries and verifying historical patterns, the final optimized model parameters were retrained on the **entire dataset (including the 2022 test set)**. 

<img width="839" height="393" alt="image" src="https://github.com/user-attachments/assets/7ece3637-5354-4c33-8300-21ffbedf2257" />




