---

## Cyclone Preheater Data Analysis Report

## Data Cleaning Summary

### 1. Initial Data Overview
- The dataset originally contained 378,719 records across 6 sensor variables.
- Data was collected every 5 minutes, covering the period from January 1, 2017 to August 7, 2020.

### 2. Duplicate and Missing Data Handling
- Duplicate rows found: 3,087  
  These were likely repeated time entries and were removed to ensure each timestamp is unique.
- Missing values (NaNs):  
  Each of the 6 columns had exactly 1 missing value (total affected rows: 1).  
  These rows were dropped to prevent issues during analysis and modeling.

| Column Name                  | Missing Values |
|------------------------------|:-------------:|
| Cyclone_Inlet_Gas_Temp       |       1       |
| Cyclone_Material_Temp        |       1       |
| Cyclone_Outlet_Gas_draft     |       1       |
| Cyclone_cone_draft           |       1       |
| Cyclone_Gas_Outlet_Temp      |       1       |
| Cyclone_Inlet_Draft          |       1       |

### 3. Final Cleaned Dataset
- Final shape after cleaning: 375,351 rows × 6 columns
- All rows are unique and contain complete sensor data.
- The dataset is now free of duplicates and missing values.

### Why This Matters
- No repeated measurements: Ensures trends and patterns are not distorted.
- No missing values: Prevents errors in calculations and machine learning algorithms.
- Reliable records: All data is trustworthy for feature engineering and anomaly detection.

In summary:  
The data cleaning process guarantees that the dataset is accurate, consistent, and ready

---

## Feature Engineering

In this step, we created new features to help the model and ourselves better understand the cyclone preheater process and detect unusual behavior.

**What was done:**

1. **Domain-specific features:**  
   - Calculated temperature and draft differences, such as the difference between inlet and outlet gas temperatures, and between different draft measurements.
   - Computed temperature efficiency as the ratio of the temperature drop to the inlet temperature.

2. **Rolling statistics:**  
   - For each column, calculated the rolling mean and rolling standard deviation over 24 hours (288 data points) and 6 hours (72 data points). This helps capture local trends and variability in the data.

3. **Z-scores:**  
   - For each value, calculated how many standard deviations it is away from its 24-hour rolling mean. High absolute z-scores indicate unusual or potentially anomalous values.

4. **Rate of change:**  
   - Computed the percentage change for each variable over a 1-hour period (12 data points). This helps identify sudden jumps or drops.


**Why this matters:**  
These engineered features provide more context and highlight abnormal patterns, making it easier for both statistical methods and machine learning models to detect anomalies in

---

## Exploratory Data Analysis (EDA) Report with Metrics

---

### 1. Univariate Analysis: Distribution of Numerical Features

We plotted histograms with KDE for each main feature to understand their distributions. Here are some key statistics:

| Feature                   | Mean  | Std Dev | Skewness | Kurtosis | Notes                                             |
|---------------------------|-------|---------|----------|----------|---------------------------------------------------|
| Cyclone_Inlet_Gas_Temp    | ~870  | ~160    | 0.3      | 3.8      | Slight right skew, bimodal structure               |
| Cyclone_Material_Temp     | ~850  | ~180    | 0.6      | 4.1      | Right skew, sharp peaks                            |
| Cyclone_Gas_Outlet_Temp   | ~880  | ~170    | 0.2      | 3.9      | Mostly normal, some heavy tails                    |
| Temp_Diff_Inlet_Outlet    | ~0    | ~50     | 0.0      | 9.5      | Sharp peak at 0, high kurtosis (many outliers)     |
| Draft_Diff_Inlet_Outlet   | ~0    | ~30     | 0.1      | 5.3      | Slight skew, heavy tails                           |

**Insights:**  
- High kurtosis (>3) means heavy tails and more outliers.
- Some features show multiple peaks, suggesting different operating modes.

---

### 2. Outlier Detection: Boxplot Analysis

Boxplots were used to visualize outliers. The percentage of outliers (using IQR) for each feature is:

| Feature                     | Outliers (%) | Skewness | Notes                                   |
|-----------------------------|--------------|----------|-----------------------------------------|
| Cyclone_Inlet_Gas_Temp      | ~2.5%        | 0.3      | Some high outliers                      |
| Cyclone_cone_draft          | ~4.3%        | -0.2     | Outliers on both ends                   |
| Temp_Diff_Material_Outlet   | ~5.1%        | 0.0      | Symmetric, high kurtosis                |
| Draft_Diff_Inlet_Outlet     | ~3.8%        | 0.1      | Balanced, heavy tails                   |
| Cyclone_Gas_Outlet_Temp     | ~2.2%        | 0.2      | Mild skew, consistent spread            |

**Insights:**  
- Most features have 2–5% outliers, often due to process changes or anomalies.

---

### 3. Correlation Analysis

The Pearson correlation matrix shows strong relationships:

| Feature Pair                                        | Correlation (r) |
|-----------------------------------------------------|-----------------|
| Cyclone_Inlet_Gas_Temp ↔ Cyclone_Gas_Outlet_Temp    | 0.97            |
| Cyclone_Material_Temp ↔ Cyclone_Gas_Outlet_Temp     | 0.94            |
| Cyclone_Gas_Outlet_Temp ↔ Temp_Diff_Material_Outlet | 0.92            |
| Temp_Diff_Inlet_Outlet ↔ Cyclone_Inlet_Gas_Temp     | 0.91            |
| Temperature_Efficiency ↔ Cyclone_Gas_Outlet_Temp    | 0.76            |

**Insights:**  
- High correlations suggest some features are redundant.
- Rolling statistics remain highly correlated with their base variables.

---

### 4. Pairplot of Top Correlated Features

Scatterplots between highly correlated features show clear linear trends and tight clusters, confirming the heatmap findings.

**Insights:**  
- Points align along diagonals, showing strong collinearity.
- Some spread in clusters may indicate non-linear effects in certain ranges.

---

### 5. Temporal Trends: Rolling Mean Analysis

A 24-hour rolling mean of Cyclone_Gas_Outlet_Temp was plotted:

| Metric            | Value/Observation                              |
|-------------------|------------------------------------------------|
| Rolling window    | 24 hours                                       |
| Mean level        | ~870–890°C                                     |
| Drops below 100°C | Recurring, likely shutdown periods             |
| Anomalous peaks   | >1300°C (rare), possible sensor spikes         |

**Insights:**  
- Rolling mean smooths out short-term noise, showing baseline and cycles.
- Recurring drops indicate downtime or cleaning cycles.
- Peaks and troughs outside normal range are flagged as anomalies.

---

### Summary Table of Key Metrics

| Aspect           | Description             | Example Metric                             |
|------------------|------------------------|--------------------------------------------|
| Spread           | Standard Deviation      | Cyclone_Material_Temp ~180°C               |
| Skewness         | Distribution Asymmetry  | Cyclone_Material_Temp = 0.6                |
| Kurtosis         | Peakiness/Tail Weight   | Temp_Diff_Inlet_Outlet = 9.5               |
| Outliers         | % outside IQR bounds    | Cyclone_cone_draft = 4.3%                  |
| Correlation      | Pearson's r             | Inlet ↔ Outlet Temp = 0.97                 |
| Rolling Mean Use | Time smoothing          | 24-hour window on Outlet Temp              |

---

### Conclusion

This EDA shows:
- Strong relationships between key temperature and draft features.
- Presence of outliers and multimodal behavior, indicating operational variability.
- Consistent temporal trends with cycles and disruptions.
- Clear feature correlations, supporting further feature engineering and modeling.

Let me know if you need a summary for stakeholders, modeling recommendations, or next

---

## Anomaly Detection and Feature Importance Report

---

### 1. Feature Engineering & Selection

To enhance the model’s sensitivity to abnormal behavior, features were derived from domain signals using:

- **Z-scores**: Capture deviation from a moving average (`*_zscore_24h`) to highlight statistical outliers.
- **Rate of Change**: First derivative of time-series features (`*_rate_change`) to detect sudden shifts.
- **Derived Differences**: Physical relationships (e.g., `Temp_Diff_Inlet_Outlet`, `Temperature_Efficiency`).

**Total features used for detection**: 18  
Categories: 9 z-score, 8 rate_change, 1 derived efficiency metric

---

### 2. Feature Scaling

- All features were standardized using `StandardScaler` to zero mean and unit variance.
- This step is essential for Isolation Forest, which is sensitive to feature scales.

---

### 3. Isolation Forest: Anomaly Detection (Base Model)

- Initial model: `n_estimators=200`, `contamination=0.05`, `random_state=42`
- Anomaly contamination rate: 5%
- Prediction mapping: `-1 → Anomaly (1)`, `1 → Normal (0)`
- Output column: `iso_anomaly`

---

### 4. Model Optimization: Grid Search with TimeSeriesSplit

A grid search was performed over:

```python
param_grid = {
    'n_estimators': [100, 200],
    'max_samples': ['auto', 0.8],
    'contamination': [0.02, 0.05, 0.1]
}
```

- Cross-validation: `TimeSeriesSplit(n_splits=3)`
- Scoring metric: `neg_mean_squared_error`

**Best Model Selected**:  
`n_estimators=200`, `max_samples=auto`, `contamination=0.05`  
Output column: `iso_tuned_anomaly` (`0 = normal`, `1 = anomaly`)

---

### 5. Model Explainability: SHAP Feature Importance

SHAP (SHapley Additive exPlanations) was used to interpret the model:

| Rank | Feature                                 | Mean SHAP Value | Description                                               |
|------|-----------------------------------------|-----------------|-----------------------------------------------------------|
| 1    | Cyclone_Material_Temp_rate_change       | 0.82            | Most influential — detects sudden shifts in material temp |
| 2    | Cyclone_Gas_Outlet_Temp_zscore_24h      | ~0.16           | Measures deviation from normal gas outlet temps           |
| 3    | Temperature_Efficiency                  | ~0.14           | Highlights heat efficiency outliers                       |
| 4    | Temp_Diff_Material_Outlet_rate_change   | ~0.13           | Detects imbalance across material temp zones              |
| 5    | Cyclone_Inlet_Draft_zscore_24h          | ~0.12           | Captures abnormal draft pressure at inlet                 |

**Key Takeaways:**

- Rate-based features are the most important, indicating abrupt changes are key indicators of anomalies.
- Z-score features help identify longer-term deviations.
- The efficiency variable is consistently influential.

---

### 6. Statistical Benchmark: Z-Score Anomaly Flagging

A rule-based method was also applied:

- Metric: Mean z-score across all z-score features per row
- Threshold: `|z| > 3` is flagged as statistically abnormal
- Columns:  
  - `zscore_mean`: Mean deviation across all z-score features  
  - `stat_anomaly`: `1 = anomaly`, `0 = normal` (if mean z-score > 3)

This baseline complements model predictions and offers a transparent sanity check.

---

## Summary: Key Outcomes

| Aspect              | Highlights                                                      |
|---------------------|----------------------------------------------------------------|
| Model Type          | Isolation Forest (unsupervised anomaly detection)              |
| Best Parameters     | n_estimators=200, contamination=0.05, max_samples=auto         |
| Top Feature         | Cyclone_Material_Temp_rate_change                              |
| Model Insights      | Change rates and z-scores are dominant predictors of anomalies |
| Baseline            | Statistical z-score threshold method aligns with model behavior|

---

---

## 🔍 Comparative Evaluation of Anomaly Detection Methods

---

### 1. Visual Analysis: Gas Outlet Temperature with Anomalies

The interactive Plotly chart displays **Cyclone Gas Outlet Temperature** over time, annotated with anomalies detected by two approaches:

- **Red markers**: Anomalies flagged by the **Isolation Forest** model.
- **Blue markers**: Anomalies identified by the **z-score-based statistical method**.
- **Overlapping points** (appear purple): Agreement between both methods.

This visualization demonstrates that most anomalies are detected by Isolation Forest, while the statistical method flags only a few, mostly extreme, deviations.

---

### 2. Method Agreement Analysis

A categorical breakdown quantifies the overlap and differences between the two methods:

| Detection Category   | Count   | Description                           |
|---------------------|---------|---------------------------------------|
| **None**            | 356,579 | No anomaly detected by either method  |
| **Isolation Only**  | 18,716  | Detected **only** by Isolation Forest |
| **Both**            | 52      | Detected by **both** methods          |
| **Statistical Only**| 4       | Detected **only** by z-score method   |

> **Interpretation**:
> - **Isolation Forest** captures the vast majority of anomalies, including subtle or nonlinear deviations.
> - The **Statistical Method** (`|z-score| > 3`) is highly conservative, flagging only the most extreme points.
> - **Overlap** is minimal, indicating statistical extremes are a small subset of model-detected anomalies.

---

### 3. Key Observations

- **Isolation Forest** is more **sensitive**, detecting both abrupt and irregular fluctuations.
- **Statistical anomalies** are rare and highlight only the most **extreme deviations**.
- Low agreement (52 points) suggests a **hybrid or consensus approach** could be valuable for operational deployment.

---

### ✅ Summary: Model Comparison Insights

| Aspect                      | Isolation Forest                              | Statistical Method               |
|-----------------------------|-----------------------------------------------|----------------------------------|
| Detection Volume            | High (18,768 total)                           | Very Low (56 total)              |
| Method Sensitivity          | High (captures subtleties)                    | Low (requires extreme deviation) |
| Agreement with other method | Only 0.3% overlap                             | N/A                              |
| Visual Impact               | Highlights transient and systematic anomalies | Focuses on extreme outliers      |

---

Would you like to see **anomaly case studies** (detailed drill-downs), or move on
---

### 6. Key Findings

- Most of the data follows regular patterns, but there are some points where the temperature or pressure behaves unusually.
- Some anomalies are detected by both methods, which means they are likely to be real process issues.
- SHAP analysis shows which features (like sudden drops or spikes in temperature) are most important for detecting anomalies.

---

### 7. Conclusion

By cleaning the data, engineering useful features, and using both statistical and machine learning methods, we can reliably detect and explain unusual events in the cyclone preheater process. The visualizations make it easy to spot and investigate these events, helping engineers maintain a stable and efficient process.
