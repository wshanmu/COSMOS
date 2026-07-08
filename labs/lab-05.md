---
layout: page
title: "Lab 5: Linear Regression and KNN Regression"
description: DS lab on home appliance energy prediction from IoT sensor data.
parent: Labs
nav_order: 5
---

# Lab 5: Linear Regression and KNN Regression **DS**{: .label .label-ds }

Date: July 10  
Time: 1:00-5:00 PM  
Dataset: Appliance energy prediction  
TA: Jianhao Huang

## Goals

By the end of this lab, you should be able to:

- use `pandas` slicing and exploratory plots to inspect sensor-style data;
- explain the difference between a feature matrix `X` and target vector `y` in `scikit-learn`;
- fit simple and multiple linear regression models;
- evaluate regression models with MAE, RMSE, and R²;
- use KNN regression with feature scaling;
- compare interpretability and prediction quality for linear regression and KNN.

## Materials

Download the Lab 5 notebook folder from the password-protected Box materials folder. The folder includes the notebook and `energydata_complete.csv`.

[Open notebook lab materials on Box](https://ucla.app.box.com/folder/397751133396){: .btn .btn-primary }

Ask Eric for the password.

Open the notebook from the downloaded Lab 5 folder so the relative path to `energydata_complete.csv` works.

## Dataset

The lab uses the appliance energy prediction dataset. Each row is a 10-minute window from a home with indoor temperature sensors, indoor humidity sensors, outdoor weather variables, and light measurements.

The target variable is:

- `Appliances`: appliance energy use in Wh for the 10-minute window.

The prediction problem is intentionally realistic. Temperature, humidity, light, and weather sensors only indirectly measure human activity, so the models should improve over simple baselines but will not be perfect.

## Four-Hour Plan

### 1:00-1:30 PM - Setup and Data Inspection

Open the notebook, verify that Python imports work, and load `energydata_complete.csv`. Check the number of rows, columns, date range, missing values, and target distribution.

Checkpoint questions:

- What does one row represent?
- What is the target variable?
- Which columns are physical sensor measurements?

### 1:30-2:10 PM - Pandas Review and Exploratory Plots

Use the small in-notebook sensor table to review `.iloc` and `.loc`, then inspect the real energy dataset with summary statistics and plots.

Deliverable:

- one plot showing appliance energy use over time;
- one short observation about why appliance energy is difficult to predict.

### 2:10-2:50 PM - Correlation and Train/Test Split

Compute correlations between sensor variables and `Appliances`. Add simple time features such as hour of day and day of week, then split the data into training and testing sets.

Discussion questions:

- Which features have the strongest relationship with appliance energy?
- Are the correlations large enough to expect perfect prediction?
- Why might a chronological split be useful in a real deployment?

### 2:50-3:35 PM - Linear Regression

Fit a one-feature linear regression model, then fit a multiple linear regression model using all physical sensor and time features.

Compare the models using MAE, RMSE, and R².

Checkpoint questions:

- Does using more features improve the test metrics?
- Why are raw coefficients hard to compare when features use different units?
- What does a low or moderate R² tell us about this sensing problem?

### 3:35-4:30 PM - KNN Regression

Fit KNN regression with a `StandardScaler` pipeline. Test several values of `k`, plot train and test R², and compare KNN with the linear models.

Deliverable:

- a table of metrics for at least three KNN values;
- a short explanation of overfitting when `k = 1`;
- your recommended model for a smart-home energy dashboard.

### 4:30-5:00 PM - Checkoff and Reflection

Each group should show the TA:

1. the dataset loaded successfully;
2. the target distribution and one time-series plot;
3. simple and multiple linear regression metrics;
4. KNN metrics for multiple `k` values;
5. a short explanation of why IoT energy prediction is noisy.

## Final Questions

Answer these briefly in your group notes:

1. What information about the home is missing from this sensor dataset?
2. Why does KNN need feature scaling?
3. Which model performed best on your test set?
4. Which model would be easiest to explain to a nontechnical user?
5. What additional IoT sensors could improve appliance energy prediction?
