# Big Data for Optimal Police Force Scheduling

## High-level problem statement

The NYPD serves over 8 million people with fewer than 35,000 officers, which makes smart scheduling essential. The NYPD has to stretch limited personnel across dozens of precincts while keeping up with constantly shifting public safety demands. Arrests and traffic crashes are two of the most common incident types requiring police response. We want to investigate whether they cluster around specific neighborhoods, times of day, and days of the week. If departments don't account for such patterns, the consequences can be slower response times, wasted resources, and gaps in public safety coverage.

This project will analyze two large public datasets: NYPD Arrest Data and NYC Motor Vehicle Collision Data. Our ultimate goal is to find patterns in when and where police-related incidents tend to occur. Using distributed data processing and machine learning, we will build predictive models that estimate future demand across different geographic areas and time windows. The end product is a scalable analytics pipeline that can help inform staffing decisions, flag high-demand periods and map out incident hotspots.

## Why This Application Domain Was Chosen

NYC Open Data provides real, recent, and large datasets, perfect for applying big data techniques. The arrest dataset contains about 279,000 rows and 19 columns, while the vehicle collisions dataset has over 2.25 million rows and 29 columns. The tables provide rich geographic, temporal, and demographic attributes. Both were recently updated, allowing the analysis to be an up-to-date reflection of modern trends. As the datasets continue to be regularly updated, the work will prove to be useful far into the future. Furthermore, this issue isn't just a toy problem. Better analysis could lead to better policy, processes, and safer communities.

## Stakeholders / Potential Users

A few different groups could make use of what our project produces:
NYPD and city agencies: to make more informed decisions about officer scheduling and deployment
City government and public safety officials: to support staffing policy and emergency response planning
Urban planners and transportation authorities: to understand how crime and crash patterns interact with infrastructure and resource needs
Emergency response coordinators: to anticipate high-demand windows and improve inter-agency coordination
Public safety researchers and policy analysts: to study urban safety trends at scale using real-world data

## Machine Learning Problem Statements

We will pursue the following three machine learning tasks, mainly using LightGBM because of its efficiency for large-scale data:
  1. **Clustering**: Discover Spatial and Temporal Patterns (Exploratory) 
This is an unsupervised task. We will use K-Means to group precincts and time periods by their incident profiles before feeding those cluster labels in as features for the LightGBM models. This reveals natural groupings in the data, such as precincts that behave similarly across the week.
  2. **Regression**: Predict Incident Volume by Precinct and Time 
The primary task is a regression problem: given a precinct, day of week, time of day, and other available features, predict the number of incidents (arrests or crashes) expected in a given time window. We'll use LightGBM’s LGBMRegressor with a squared error objective to estimate these counts. This loss function places greater penalty on larger prediction errors, which is important because underestimating high-demand periods could result in insufficient police staffing.
  3. **Classification**: Flag High-Demand Periods 
The final task is a multi-class classification: label each precinct-time window as low, moderate, or high demand based on historical incident counts. This is clearer to shift planners and policy-makers. LightGBM supports multi-class objectives directly and provides feature importance metrics, allowing us to verify that the model is relying on meaningful predictors rather than noise.

## Data Analysis Objectives

Given those three machine learning tasks, we have six objectives in regards to what we wish to learn from the data:

**Temporal Distribution of Incidents**: Use a MapReduce job to aggregate arrest and crash records by date across the full dataset, then analyze the resulting distributions to identify which windows are consistently the busiest. Processing this at scale via Hadoop will ensure efficient processing of the entire dataset.

**Geographic Hotspot Identification**: Map incident density across NYC's precincts by running a MapReduce aggregation keyed on geographic identifiers, then visualize the output to areas with disproportionately high or persistent demand.

**Feature Importance via LightGBM**: Use LightGBM's built-in feature importance metrics to determine which variables (precinct, time of day, day of week, borough, incident type) drive predictions the most.

**Correlation Between Arrest and Crash Patterns**: Use a MapReduce join across the two datasets, keyed on precinct and time window, to examine whether areas with high arrest activity also tend to see elevated crash rates, or whether the two incident types cluster independently.

**Seasonal and Long-Term Trends**: Run a MapReduce job grouped by month and year to analyze how incident volumes shift over time, and assess whether patterns are stable enough across years for a model trained on historical data to generalize reliably to future scheduling periods.
Precinct-Level Demand Profiling: Build distinct demand profiles for individual precincts, enabling more targeted staffing recommendations rather than broad city-wide averages.

## Input → Output Specification

**Regression (Incident Volume Prediction)**
**Input**: The two primary datasets are the NYPD Arrest Data and the NYC Motor Vehicle Collision Data, both sourced from the NYC Open Data portal. The key fields drawn from these datasets include ARREST_PRECINCT and LOCATION (geographic unit), ARREST_DATE / CRASH_DATE and CRASH_TIME (temporal fields), and LATITUDE / LONGITUDE (for spatial aggregation). 

These raw records will be aggregated via MapReduce into a structured feature table keyed on (precinct, day_of_week, hour_of_day), with additional engineered fields such as month, borough, and incident_type included as categorical features.

**Output**: The predicted number of incidents (arrest or crash count) per precinct per time window. The unit of analysis is a (precinct × hour × day_of_week) cell, representing an expected incident volume that can directly inform how many officers should be scheduled for a given precinct during a given shift.

## Classification (Demand Level Labeling)

**Input**: The aggregated feature table produced by the regression pipeline, with the continuous incident count replaced by a derived categorical label. Demand thresholds (low / moderate / high) will be defined based on the count distribution computed during MapReduce preprocessing — for example, using percentile cutoffs across all precinct-time windows.

**Output**: A demand level label (low, moderate, or high) for each (precinct × hour × day_of_week) window. This gives scheduling coordinators a simple signal about expected busyness.

## Clustering (Precinct and Temporal Profiling)

**Input**: Per-precinct features computed via MapReduce and stored in HDFS, summarizing each precinct's incident profile across the full historical dataset. Key fields include average incident counts by hour, peak demand windows, and incident type mix.

**Output**: Cluster assignments for each precinct and each time period, grouping them by behavioral similarity. The labels will be fed back into the regression and classification models as engineered features.


## Resources

**LightGBM**: https://lightgbm.readthedocs.io/en/latest/ 

**NYC Open Data Motor Vehicle Collisions database**: https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95/about_data

**NYC Open Data Arrests**: https://data.cityofnewyork.us/Public-Safety/NYPD-Arrest-Data-Year-to-Date-/uip8-fykc/about_data 
