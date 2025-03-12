# Well Performance Prediction and Optimization

**Project Overview**

This project focuses on analyzing historical well production data to predict and optimize oil production while minimizing water production.  The project leverages machine learning (specifically, regression models) to understand the relationships between various downhole and surface parameters and the resulting oil and water volumes. The ultimate goal is to provide insights into optimal operating conditions (specifically, `ON_STREAM_HRS` and `AVG_CHOKE_SIZE_P`) to maximize oil production and minimize unwanted water production.

**Data Source**

The data used in this project comes from the "Downhole Well Production" 
*   **File:** `Well production analysis data.csv`
*   **Assumed Source:** https://www.kaggle.com/datasets/sobhanmohammadids/drilling-well-production-data/data

**Workflow**

The project follows a structured workflow:

1.  **Data Loading and Preprocessing:**
   
2.  **Data Splitting (Per Well):**
   
3.  **Exploratory Data Analysis (EDA):**
    *   Creates scatter plots to visualize relationships:
        *   `BORE_OIL_VOL` vs. `ON_STREAM_HRS`
        *   `BORE_OIL_VOL` vs. `AVG_CHOKE_SIZE_P`
        *   `BORE_OIL_VOL` vs. `AVG_DOWNHOLE_PRESSURE`
        *   `BORE_OIL_VOL` vs. `AVG_DOWNHOLE_TEMPERATURE`
    *   Calculates and displays a correlation matrix (heatmap) of the key numerical features. This helps identify potential multicollinearity and strong predictors.

4.  **Model Building:**
    *   **Data Scaling:** Standardizes the numerical features using `StandardScaler`. This is essential for models like Linear Regression that are sensitive to feature scaling.
    *   **Train/Test Split:** Splits the data into training (75%) and testing (25%) sets using `train_test_split`.  
    *   **Regression Models:** Trains three regression models:
        *   `LinearRegression`
        *   `RandomForestRegressor` (with `n_estimators=100`)
        *   `GradientBoostingRegressor`
    *   **Model Evaluation:** Calculates and prints the Mean Squared Error (MSE) and R-squared score for each model on the test set.
    *  **Feature importance:** Uses `RandomForestRegressor` and displays feature importance.

5.  **Optimization Function (`optimize_production`):**
    *   This is the core of the project.  The function takes a well-specific DataFrame, target variables (`BORE_OIL_VOL`, `BORE_WAT_VOL`), a list of features, and an objective (currently set to "maximize_oil_minimize_water") as input.
    *   **Separate Models for Oil and Water:** Critically, it trains *separate* `RandomForestRegressor` models for oil and water production.  This allows the optimization to consider the trade-off between maximizing oil and minimizing water.
    *   **Grid Search:** Performs a grid search over a range of `ON_STREAM_HRS` and `AVG_CHOKE_SIZE_P` values (using `np.linspace` to create a grid). This is a simple but effective optimization technique.  *Important Note:* The `grid_density` parameter controls the granularity of the search.
    *   **Objective Function:**  For each combination of `ON_STREAM_HRS` and `AVG_CHOKE_SIZE_P`, it:
        *   Creates an input array with the mean values for the other features (downhole pressure, temperature, etc.).
        *   Predicts oil and water production using the trained models.
        *   Calculates an `objective_value` which is currently defined as `predicted_oil - predicted_water`. This means the goal is to maximize the *difference* between oil and water production. *Important Note:* This objective function could be customized. For example, you could add weights to prioritize oil production or introduce a penalty for excessive water production.
        *   Tracks the best `ON_STREAM_HRS` and `AVG_CHOKE_SIZE_P` values that maximize the `objective_value`.
    *   **Returns:**  The function returns the optimal `ON_STREAM_HRS`, `AVG_CHOKE_SIZE_P`, the corresponding predicted oil production, and the minimum water production.

6.  **Per-Well Optimization:**
    *   Iterates through each well-specific DataFrame in the `well_dataframes` dictionary.
    *   Calls the `optimize_production` function for each well.
    *   Prints the optimization results (optimal parameters and predicted production) for each well.

**Key Findings and Results:**

*   **Model Performance:** The Random Forest model significantly outperforms Linear Regression in terms of both MSE and R-squared, indicating a non-linear relationship between the features and oil production. GradientBoostingRegressor also performs well, but Random Forest is slightly better.
*   **Feature Importance:** The feature importance plot from the Random Forest model shows that `AVG_WHP_P` (average wellhead pressure) and `AVG_WHT_P` (average wellhead temperature) are the most important predictors of oil production, followed by `AVG_CHOKE_SIZE_P` and `ON_STREAM_HRS`.
*   **Optimization Results:** The `optimize_production` function provides well-specific recommendations for `ON_STREAM_HRS` and `AVG_CHOKE_SIZE_P` to maximize oil production while minimizing water production.  The specific optimal values vary for each well, as expected.


**Data Dictionary (`Well production analysis data.csv`):**

| Column Name              | Description                                                        | Data Type  |
| :------------------------ | :----------------------------------------------------------------- | :--------- |
| DATEPRD                   | Date of production                                                 | datetime   |
| NPD_WELL_BORE_NAME        | Name of the wellbore                                              | object     |
| ON_STREAM_HRS             | Number of hours the well was on stream (producing)                 | float      |
| AVG_DOWNHOLE_PRESSURE     | Average downhole pressure                                          | float      |
| AVG_DOWNHOLE_TEMPERATURE  | Average downhole temperature                                       | float      |
| AVG_DP_TUBING             | Average differential pressure in the tubing                       | float      |
| AVG_ANNULUS_PRESS         | Average annulus pressure                                           | float      |
| AVG_CHOKE_SIZE_P          | Average choke size (percentage)                                    | float      |
| AVG_WHP_P                 | Average wellhead pressure                                           | float      |
| AVG_WHT_P                 | Average wellhead temperature                                        | float      |
| DP_CHOKE_SIZE             | Differential pressure across the choke                             | float      |
| BORE_OIL_VOL              | Volume of oil produced (target variable)                           | float      |
| BORE_GAS_VOL              | Volume of gas produced                                              | float      |
| BORE_WAT_VOL              | Volume of water produced                                            | float      |
| BORE_WI_VOL               | Volume of water injected                                           | float      |
| FLOW_KIND                 | Type of flow (e.g., "production", "injection")                     | object     |
| pred_L_reg                | Predicted BORE_OIL_VOL from Linear Regression                      | float      |
| pred_RFR                | Predicted BORE_OIL_VOL from Random Forest Regressor              | float      |
| gbr_pred                | Predicted BORE_OIL_VOL from GradientBoostingRegressor        | float      |

