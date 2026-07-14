---
layout: page
title: "Lab 8: Train/Validation/Test Split and Polynomial Regression"
description: DS lab on train/validation/test splits, polynomial regression, and multiple regression with transformed features.
parent: Labs
nav_order: 8
---

# Lab 8: Train/Validation/Test Split and Polynomial Regression **DS**{: .label .label-ds }

Date: July 16  
Time: 1:00-5:00 PM  
TA: Jianhao Huang

## Overview

Today we will build on linear regression by asking a deeper modeling question: how do we know whether a model is actually useful on data it has not seen before?

The lab focuses on train/validation/test splits, polynomial regression, transformed features, and multiple regression. You will first revisit a taxi pickup dataset to compare a straight-line model with a polynomial model. Then you will work with English Premier League player data to practice stratified splitting, grouping, feature engineering, dummy variables, and interpretation of regression coefficients.

## Goals

By the end of this lab, you should be able to:

- explain why we keep separate training, validation, and testing data;
- use `train_test_split()` and understand when stratified splitting is useful;
- compare model performance on held-out data using R²;
- use `PolynomialFeatures` to fit a curved relationship with a linear model;
- use feature scaling when polynomial features become large;
- build multiple regression models with transformed variables such as age² and `log2(page_views)`;
- convert categorical variables into dummy variables with `pd.get_dummies()`;
- interpret model coefficients carefully, especially for transformed and dummy variables.

## Materials

Download the Lab 8 notebook folder from the password-protected Box materials folder:

[Open notebook lab materials on Box](https://ucla.app.box.com/folder/397751133396){: .btn .btn-primary }

Ask Eric for the password.

Open the folder:

```text
Lab_08_Train_Validation_Test_Split_and_Polynomial_Regression
```

The folder should include:

| File | Purpose |
| --- | --- |
| `COSMOS_Lab_08_Train_Validation_Test_Split_and_Polynomial_Regression.ipynb` | Main Lab 8 notebook. |
| `dataset_1.txt` | Taxi pickup dataset used for linear and polynomial regression. |
| `league_data.txt` | English Premier League player dataset used for multiple regression. |

Open the notebook from inside the downloaded Lab 8 folder so the relative paths to `dataset_1.txt` and `league_data.txt` work.

## Notebook Flow

Work through the notebook in order:

1. Review train/validation/test split terminology and the difference between inference and prediction.
2. Fit a simple linear regression model to taxi pickup counts by time of day.
3. Refit the taxi model using polynomial features and compare held-out R².
4. Try feature scaling with `StandardScaler`.
5. Load and inspect the football dataset.
6. Use stratified train/test splitting by `region`, fixing any missing-value issue that prevents the split.
7. Use `.groupby()` and `.agg()` to summarize market value, page views, and fantasy points by position and club type.
8. Build a multiple regression model for `market_value` using numeric, polynomial, log-transformed, and categorical features.
9. Recode `position_cat` with dummy variables and interpret the new coefficients.
10. Sketch a forward-selection algorithm using set symmetric difference.

## Main Lab Tasks

### Taxi Pickup Regression

Use `dataset_1.txt` to model taxi pickup count as a function of time of day.

You should complete these tasks:

1. Load the cab data and split it into training and testing sets.
2. Fit a simple `sklearn.linear_model.LinearRegression` model using `TimeMin`.
3. Refit the same linear model using `statsmodels` and compare the R² values.
4. Use `PolynomialFeatures` to create polynomial features such as `x`, `x^2`, and `x^3`.
5. Fit the polynomial regression model and calculate its test-set R².
6. Make a residual plot for the polynomial model and explain what the residuals show.
7. Try scaling the polynomial features with `StandardScaler`.

### Football Multiple Regression

Use `league_data.txt` to predict a player's `market_value`.

You should complete these tasks:

1. Load the football dataset and inspect column types, shape, and summary statistics.
2. Create a stratified train/test split using `region`.
3. Identify and handle the missing-value issue that prevents stratified splitting from working at first.
4. Use `.groupby()` to compare market value, page views, and fantasy points across positions and club categories.
5. Build the model:

   ```text
   market_value ~ fpl_points + age + age^2 + log2(page_views) + new_signing + big_club + position_cat
   ```

6. Add the `age_squared` and `log_views` transformed features.
7. Add a constant term for `statsmodels`.
8. Fit the model with `OLS` and evaluate held-out R² with `r2_score`.
9. Interpret the coefficients for age, age², `log2(page_views)`, and `big_club`.
10. Replace the numeric `position_cat` feature with dummy variables and interpret `position_cat_2`.

## Checkoff

Show the TA:

1. The Lab 8 notebook runs from the imports through both datasets.
2. The taxi linear model and polynomial model, including their test-set R² values.
3. A residual plot for the polynomial taxi model.
4. A successful stratified train/test split for the football dataset.
5. At least one `.groupby()` summary table for the football data.
6. The football multiple regression summary and held-out R² score.
7. Your interpretation of at least two regression coefficients.
8. The dummy-variable version of the position model.

## Final Questions

Answer these briefly in your group notes:

1. Why should the test set stay untouched until final model evaluation?
2. What changed when you used polynomial features on the taxi data?
3. Why might scaling be helpful after creating polynomial features?
4. Why is `position_cat` a poor feature if we leave it as the numbers 1, 2, 3, and 4?
5. What is the difference between using regression for prediction and using regression for inference?
