---
layout: page
title: "Lab 7: Linear Regression Big Lab"
description: DS lab on an end-to-end linear regression workflow with real datasets.
parent: Labs
nav_order: 7
---

# Lab 7: Linear Regression Big Lab **DS**{: .label .label-ds }

Date: July 14  
Time: 1:00-5:00 PM  
TA: Jianhao Huang / Shanmu Wang

## Overview

Today we will work through an end-to-end dataset workflow. The goal is not just to call `LinearRegression`, but to practice the full process:

1. load a real dataset;
2. inspect rows, columns, missing values, and summary statistics;
3. make exploratory visualizations;
4. clean and prepare features;
5. handle non-numeric columns and missing values;
6. split data into training and testing sets;
7. fit a linear regression model;
8. evaluate prediction quality with an error metric such as MSE.

## Materials

Download the Lab 7 notebook folder from the password-protected Box materials folder:

[Open notebook lab materials on Box](https://ucla.app.box.com/folder/397751133396){: .btn .btn-primary }

Ask Eric for the password.

Open the folder:

```text
Lab_07_Linear_Regression_Big_Lab
```

The folder should include:

| File | Purpose |
| --- | --- |
| `Lab7_big_lab_practice.ipynb` | Practice notebook. Run this first to review the end-to-end regression workflow. |
| `Lab07_Linear_Regression_Big_Lab.ipynb` | Main lab notebook. This is the notebook you will complete after the practice notebook. |
| `housing.csv` | Practice dataset used by the practice notebook. |
| `AB_NYC_2019.csv` | Main NYC Airbnb dataset used by the big lab. |
| `california.png` | Image used by the practice notebook. |
| `nyc.png` | Image used by the main lab notebook. |

## Notebook Order

Start with the practice notebook:

```text
Lab7_big_lab_practice.ipynb
```

Run it from top to bottom before opening the main lab. This notebook is meant to refresh the full regression pipeline and help you remember the common steps before you work more independently.

Then open the main notebook:

```text
Lab07_Linear_Regression_Big_Lab.ipynb
```

In the main lab, you will apply the same end-to-end machine learning steps to the NYC Airbnb dataset. The prediction task is to use listing features to predict Airbnb rental price.

## Main Lab Tasks

In the main notebook, you should complete these parts:

1. Load `AB_NYC_2019.csv`.
2. Inspect the data and drop columns that are not useful for the model.
3. Visualize room type, borough, reviews, location, and price patterns.
4. Create a binned `price_cat` feature for stratified splitting.
5. Handle missing values.
6. Convert non-numeric features into numeric features.
7. Split the data into training and testing sets.
8. Fit a linear regression model to predict `price`.
9. Evaluate the model using MSE and explain what the result means.

## Checkoff

Show the TA:

1. The practice notebook ran successfully.
2. The main notebook loads the NYC Airbnb dataset.
3. At least two completed visualizations.
4. The cleaned/prepared feature table.
5. The trained linear regression model and MSE result.
