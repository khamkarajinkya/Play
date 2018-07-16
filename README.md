### Effective Markup Anomaly Detection

This job detects anomalies in effective markup of order prices at the store level. Effective markup is defined as below and should typically range between 15 to 18 %

```

(actual_subtotal / cost) - 1

``` 

Instead of per order effective markup, we calculate an aggregated effective markup by taking the sum of costs and actual_subtotals for all orders falling in same time interval. Per order effective markups is much more noisy. Aggregating tends to smoothen the effective markup. This behaviour can be controlled using `agg_interval` parameter within the script. 

Anomaly detection is performed using Median Absolute Deviation and are detected as follows


```
For time points {1....t} in x

If |x_t - median(x)| >= threshold * MAD (x) ; x_t is an anomaly

MAD (x) = median(|x - median(x)|)
```

The length t of the rolling window determines the sensitivity of the detection. Larger windows with several time points tend to be more robust and less sensitive to sudden changes in effective markup. This behavior can be controlled by using the `window_interval` parameter in the script. 
 

#### Job Parameters

1. `required_date` : Datetime for which anomaly detection has to be performed, entered at commandline (default current)
2. `store_id` : Stores for which effective markup anomaly detection has to be performed (Default 1, 10 )
3. `agg_interval`: Interval over which order cost and actual_subtotal should be aggregated (Default 4)
4. `window_interval`: Length of rolling time window (Default 21 time points)

#### Job Output

Database

The job writes 1 record per store to the `effective_markup_anomaly` table in the `data-processing.public` schema per run



Slack

1. The job pushes anomalous records as slack updates to #markup-anomaly-detection 
2. The job pushes plot of effective markup vs time to #markup-anomaly-detection if any record in the past 24 hours was anomalous


#### Output Table Schema

| Column Name | Description                     |
|----------|-------------------------------|
| store_id  | Store id |
| date_utc | Datetime for which anomaly detection is performed in UTC |
| date_ct  | Datetime for which anomaly detection is performed in Central Time |
| store_name  | Store name |
| no_of_orders | number of orders in the aggregate interval used to calculate statistic |
| effective_markup | effective markup at date_utc |
| lower_bound_markup | Lower bound of effective markup at date_utc |
| upper_bound_markup | Upper bound of effective markup at date_utc |
| anomaly | Anomaly or not |
| aggregation_interval_in_hrs | Aggregation interval used in the job run |
| lookback_window_in_hrs | Lookback interval used in the job run |
| created_at | record creation timestamp |
| updated_at | record update timestamp (should be same as updated_at, as record once created does not require updation)|

#### Things to complete 

1. agg_interval and window_interval can be made store specific instead of run specific

