# Naive-Forecast  

### Introduction to Naïve Forecasting  

Forecasting methods range from highly complex models to very simple approaches. One such straightforward yet surprisingly effective method is the naïve forecast. The naïve approach assumes that the best forecast for any period is simply the value of the most recent observation. This simplicity makes it useful as a baseline model to compare against more sophisticated forecasting techniques.  

### Project Objective  

Our objective is to develop a naïve forecast for a newly defined metric, "distance per dollar," and evaluate its predictive performance using Root Mean Squared Error (RMSE).  

### Metric Definition  

distance_per_dollar = distance_to_travel / monetary_cost  

This metric reflects how much distance can be traveled per unit of monetary cost, making it a key indicator of cost-efficiency in travel.  

### Data Description  

The project uses a dataset from Uber with the following fields:  

- request_id: int – Unique identifier for each request
- request_date: datetime – Date and time when the request was made
- request_status: varchar – Status of the request (e.g., completed, canceled)
- distance_to_travel: float – Distance the client intends to travel
- monetary_cost: float – Cost associated with the travel
- driver_to_client_distance: float – Distance from the driver to the client when the request was made

### Problem Definition  

Our goal is to create a naïve forecast for the "distance per dollar" metric and evaluate its accuracy. We follow these steps:

1. Aggregate "distance to travel" and "monetary cost" at a monthly level.
2. Calculate "distance per dollar" for each month, which serves as the actual value.
3. Populate the forecasted value for each month using the previous month's actual value.
4. Evaluate the model using RMSE, defined as:

   RMSE = sqrt(mean(square(actual - forecast)))

The RMSE will be rounded to two decimal places.  

### Solution Implementation  

```postgresql
WITH monthly_aggregates AS (
    SELECT 
        EXTRACT(MONTH FROM request_date) AS month, 
        SUM(distance_to_travel) AS total_distance, 
        SUM(monetary_cost) AS total_cost
    FROM 
        uber_request_logs
    GROUP BY 
        month
),
distance_per_dollar_calc AS (
    SELECT 
        *, 
        (total_distance / total_cost) AS distance_per_dollar
    FROM 
        monthly_aggregates
    ORDER BY 
        month
),
naive_forecast AS (
    SELECT 
        *, 
        LAG(distance_per_dollar, 1) OVER (ORDER BY month) AS forecast
    FROM 
        distance_per_dollar_calc
)
SELECT 
    SQRT(AVG(POWER(distance_per_dollar - forecast, 2))) AS rmse
FROM 
    naive_forecast;

```
### Explanation of the SQL Solution  

1. CTE: monthly_aggregates

- Purpose: Aggregate data at the monthly level to calculate the total "distance to travel" and "monetary cost."
- Logic: Use EXTRACT(MONTH FROM request_date) to extract the month, and compute the total distance_to_travel and monetary_cost for each month.

2. CTE: distance_per_dollar_calc

- Purpose: Calculate "distance per dollar" for each month.
- Logic: Divide total_distance by total_cost to get distance_per_dollar and order by month.

3. CTE: naive_forecast

- Purpose: Generate the naïve forecast using the LAG window function.
- Logic: Use LAG(distance_per_dollar, 1) OVER (ORDER BY month) to get the previous month's "distance per dollar" value for forecasting.

4. Final Calculation

- Purpose: Calculate RMSE to evaluate forecast accuracy.
- Logic: Compute squared differences between distance_per_dollar and the forecasted values, then use SQRT(AVG(POWER(...))) to calculate RMSE.

### Conclusion  

This solution uses a simple SQL approach to implement a naïve forecast for the "distance per dollar" metric. By structuring the query with descriptive CTEs, we simplify data processing and improve readability. The RMSE value serves as a baseline performance metric for comparing more complex forecasting models.







