---
layout: page
title: "Lab 10: Logistic Regression"
description: DS lab on binary and multiclass logistic regression using the NYC Airbnb 2019 dataset.
parent: Labs
nav_order: 10
---

# Lab 10: Logistic Regression **DS**{: .label .label-ds }

Date: July 20  
Time: 1:00-5:00 PM  
TA: Jianhao Huang

## Overview

Today we will move from predicting continuous values to predicting categories. Using the NYC Airbnb 2019 dataset, we will classify listings by price and compare several approaches:

1. a majority-class baseline, called MLE in the notebook, that ignores the features;
2. ordinary linear regression followed by a threshold;
3. regularized Ridge and Lasso regression followed by a threshold;
4. binary logistic regression; and
5. multiclass logistic regression for five price levels.

The notebook follows the full data science workflow: define the target, split the data, inspect only the training set, clean and encode features, establish a baseline, train several models, tune them on a development set, and interpret their behavior. The testing set should remain untouched while you work through the notebook.

## Goals

By the end of this lab, you should be able to:

- distinguish a classification task from a regression task;
- construct binary and multiclass labels from a continuous variable;
- explain the different roles of training, development, and testing data;
- use a stratified split to preserve the proportion of a binary target;
- perform missing-value and outlier analysis using only the training data;
- convert categorical variables into one-hot encoded features;
- compute and interpret a simple majority-class baseline;
- use a threshold to turn a linear-regression output into a class prediction and explain the limitations of doing so;
- compare ordinary, Ridge, and Lasso regression using development-set accuracy;
- fit and evaluate `sklearn.linear_model.LogisticRegression`;
- explain how `C`, `max_iter`, `penalty`, and `solver` affect logistic regression; and
- extend binary logistic regression to a five-class prediction problem.

## Materials

Download the Lab 10 notebook folder from the password-protected Box materials folder:

[Open notebook lab materials on Box](https://ucla.app.box.com/folder/397751133396){: .btn .btn-primary }

Ask Eric for the password.

Open the folder:

```text
Lab_10_Logistic_Regression
```

The folder should include:

| File | Purpose |
| --- | --- |
| `COSMOS_Lab_10_Logistic_Regression.ipynb` | Main Lab 10 notebook. |
| `nyc_airbnb.csv` | NYC Airbnb 2019 listing data used throughout the lab. |

Open the notebook from inside the downloaded Lab 10 folder. The notebook reads `./nyc_airbnb.csv`, so keeping the notebook and CSV together is important.

## Dataset and Prediction Tasks

Each row of `nyc_airbnb.csv` describes one Airbnb listing. Available fields include its borough and neighborhood, latitude and longitude, room type, nightly price, minimum-night requirement, review information, host listing count, and yearly availability.

### Binary Task

The first prediction target is `affordable`:

| Label | Meaning |
| ---: | --- |
| `1` | Nightly price is less than \$150. |
| `0` | Nightly price is \$150 or more. |

The original `price` column is used to create this label and must then be removed from the feature matrix. Leaving it in the features would reveal the answer to the model.

### Multiclass Task

The final part replaces the binary target with five price levels. The notebook uses price cut points at \$80, \$120, \$180, and \$240 to create labels from `0` through `4`, ranging from budget to very expensive.

## Notebook Flow

Work through the notebook from top to bottom:

1. Import the data science and modeling libraries.
2. Load the Airbnb dataset, define `affordable`, rename the borough column, and remove zero-price listings.
3. Create approximately 60% training, 20% development, and 20% testing splits. The first split is stratified by `affordable`.
4. Separate the feature matrices from the target vectors so that neither `price` nor `affordable` appears in the model inputs.
5. Inspect missing values using the training set, then consistently drop `last_review` and `reviews_per_month` from all three splits.
6. Inspect the `minimum_nights` distribution and remove training listings requiring more than 30 nights.
7. Explore the remaining training features with summary statistics, histograms, and a scatter matrix.
8. Establish the notebook's majority-class baseline.
9. One-hot encode `borough` and `room_type`, remove text identifiers, and compare ordinary, Ridge, and Lasso regression as thresholded classifiers.
10. Fit binary logistic-regression models and tune their hyperparameters using development accuracy.
11. Create five price classes with `pd.cut()` and repeat the logistic-regression workflow for multiclass classification.

## Main Lab Tasks

### Part 1: Data Preparation and Training-Only EDA

Create the binary label before splitting, but do all exploratory decisions using the training data.

You should complete these tasks:

1. Remove listings whose nightly price is zero.
2. Confirm the training/development/testing proportions printed by the notebook.
3. Remove `price` and `affordable` from each feature matrix.
4. Count missing values by column.
5. Drop the two review-date/rate columns from every split after diagnosing them on training data.
6. Plot `minimum_nights` on a logarithmic count scale.
7. Restrict the training data to `minimum_nights <= 30` and keep `x_train` and `y_train` aligned.
8. Inspect the remaining numeric features and scatter matrix for outliers and collinearity.

Important: do not remove development or testing rows merely because their values look unusual. Those splits represent unseen data. Any transformation learned from training data must be applied consistently without using their labels to make decisions.

### Part 2: Majority-Class Baseline (Notebook MLE Section)

For Exercise 1, find the most common value of `affordable` in the training labels. Predict that value for every development example and report the accuracy.

This deliberately simple model answers an important question: does a learned model outperform always predicting the majority class?

### Part 3: Thresholded Linear Models

Prepare a numeric feature matrix containing:

- four one-hot columns for the five boroughs;
- `latitude` and `longitude`;
- two one-hot columns for the three room types;
- `minimum_nights`;
- `number_of_reviews`;
- `calculated_host_listings_count`; and
- `availability_365`.

After encoding and dropping the unused identifier/text columns, the notebook expects 12 feature columns.

Complete Exercises 2-5:

1. Fit an ordinary least-squares model with an intercept using `statsmodels.OLS`.
2. Predict on the development set.
3. Convert each numeric prediction to a class using a threshold of `0.5`.
4. Report development accuracy and compare it with the MLE baseline.
5. Fit Ridge and Lasso models for each listed `alpha` value and retain the best development result.
6. Plot separate histograms of training and development residuals from the best regularized linear model.
7. Discuss whether the training residuals look consistent with the assumptions of a linear model.

Remember that linear regression is being used here as a baseline classifier. Its outputs are not restricted to the probability interval from 0 to 1, which is one reason logistic regression is a more natural model for a binary target.

### Part 4: Binary Logistic Regression

Fit `sklearn.linear_model.LogisticRegression` on the same numeric features, predict the development labels, and report accuracy.

Then compare multiple settings of:

- `C`: the inverse regularization strength; it must be greater than zero, and smaller values mean stronger regularization;
- `max_iter`: the maximum number of solver iterations;
- `penalty`: for example, L1 or L2 regularization when supported by the selected solver; and
- `solver`: the numerical algorithm, which must be compatible with the penalty and classification setting.

Use only development performance to choose the best setting. Record the best model, accuracy, and coefficient vector. Do not assume that a larger `C` or a larger `max_iter` must improve unseen-data accuracy.

The notebook's link to the [`LogisticRegression` documentation](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html) is useful when checking compatible parameter combinations.

### Part 5: Multiclass Logistic Regression

Use the provided `pd.cut()` cells to construct the five price-level labels for training and development data. Then:

1. fit multiclass logistic-regression models for the provided `C` and iteration settings;
2. track the best development accuracy and model;
3. print the learned coefficient dimensions;
4. compare multiclass performance with the binary task; and
5. suggest additional features or transformations that could improve the result.

Possible ideas include one-hot encoding individual neighborhoods, scaling latitude and longitude, using a `Pipeline`, and performing cross-validation on the training data.

## Parameter Reminders

- Use `max_iter`, not `max_iterations`, when constructing a scikit-learn logistic-regression model.
- `C` must be positive; do not include zero in a `C` search.
- Small `C` means stronger regularization, while large `C` means weaker regularization.
- Some penalty/solver combinations are invalid. Check the documentation before starting a large search.
- After one-hot encoding, verify that training, development, and testing data have the same columns in the same order.
- The notebook creates a test set but focuses on development-set model comparison. Keep the test set untouched unless the TA explicitly asks for one final evaluation.
- The notebook numbering jumps from Exercise 6 to Exercise 8; no Exercise 7 response is required.

## Checkoff

Show the TA:

1. The notebook loads `nyc_airbnb.csv` and constructs the 60/20/20 data split.
2. `price` and `affordable` are absent from the feature matrices.
3. The missing-value audit and the two `minimum_nights` histograms.
4. The cleaned training data and 12-column encoded feature matrix.
5. The majority-class MLE prediction and development accuracy.
6. The ordinary linear-regression development accuracy using the `0.5` threshold.
7. The Ridge/Lasso parameter comparison and two residual histograms.
8. The binary logistic-regression accuracy and best hyperparameter setting.
9. The five-class target, best multiclass model, and development accuracy.
10. A short explanation of one feature or preprocessing change you would try next.

## Final Questions

Answer these briefly in your group notes:

1. Why is the MLE majority-class result an important baseline?
2. Why must `price` be removed before predicting `affordable` or `price_level`?
3. Why do we one-hot encode borough and room type instead of assigning arbitrary integers?
4. What can go wrong when ordinary linear regression is used for binary classification?
5. How does changing `C` alter the strength of logistic-regression regularization?
6. Why should hyperparameters be selected using development data rather than test data?
7. Why might the five-class problem be harder than the binary problem?
8. In a high-stakes classification problem, why might accuracy alone be insufficient?
