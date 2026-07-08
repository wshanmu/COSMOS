---
layout: page
title: "Lab 2: Data Analysis and Visualization"
description: DS lab on exploratory data analysis with pandas, matplotlib, seaborn, Titanic data, and heart disease data.
parent: Labs
nav_order: 2
---

# Lab 2: Data Analysis and Visualization **DS**{: .label .label-ds }

Date: July 7  
Time: 1:00-5:00 PM  
Dataset: Titanic passengers and heart disease data  
TA: Jianhao Huang

## Goals

By the end of this lab, you should be able to:

- choose an appropriate plot type for a basic exploratory data analysis question;
- use `seaborn` and `matplotlib` to make histograms, KDE plots, boxplots, scatterplots, count plots, and pair plots;
- use `pandas` operations such as `.drop()`, boolean filtering, `.isna()`, `.describe()`, `.value_counts()`, and built-in plotting;
- compare categorical and numerical variables visually;
- inspect missing values and basic dataset structure before making conclusions;
- use plots to answer short data analysis questions about a real medical dataset.

## Materials

Download the Lab 2 notebook folder from the password-protected Box materials folder. The folder includes the notebook and `heart_disease.csv`.

[Open notebook lab materials on Box](https://ucla.app.box.com/folder/397751133396){: .btn .btn-primary }

Ask Eric for the password.

Open the notebook from the downloaded Lab 2 folder so the relative path to `heart_disease.csv` works. The Titanic dataset is loaded directly through `seaborn`, so it does not need a separate file.

## Datasets

This lab uses two datasets for two different purposes.

### Titanic Dataset

The Titanic dataset is used as a guided visualization warm-up. Each row represents a passenger, with variables such as age, fare, class, sex, survival status, and deck.

Useful questions:

- What ages were represented among passengers?
- How were ticket class, age, and fare related?
- Which variables are categorical, and which are numerical?
- Where are missing values present?

### Heart Disease Dataset

The heart disease dataset is used for the group exercise. Each row represents a patient record with clinical variables such as age, sex, chest pain type, resting blood pressure, cholesterol, maximum heart rate, exercise-induced angina, and a diagnosis variable.

This dataset is for data visualization practice only. Do not interpret the lab as medical advice or diagnostic guidance.

## Four-Hour Plan

### 1:00-1:20 PM - Setup and Notebook Imports

Open the notebook, select the correct Python kernel, and run the import cells.

Checkpoint:

- `numpy`, `pandas`, `matplotlib`, `seaborn`, and `sklearn` import successfully;
- plots display inside the notebook;
- the Titanic dataset loads with `sns.load_dataset("titanic")`.

### 1:20-1:50 PM - Inspect and Clean the Titanic Data

Use the Titanic dataset to practice basic dataset inspection:

- display column names;
- inspect dataset information with `.info()`;
- drop columns that are not needed for the first visualizations;
- count missing values in the `deck` column.

Checkpoint questions:

- What does one row represent?
- Which columns are numerical?
- Which columns are categorical?
- Why should we check missing values before plotting?

### 1:50-2:35 PM - Histograms and Density Plots

Use histograms and KDE plots to inspect one variable at a time.

Notebook examples include:

- age distribution with `seaborn`;
- age distribution with `matplotlib`;
- custom histogram colors;
- KDE bandwidth comparison;
- fare distribution.

Deliverable:

- one histogram or KDE plot for passenger age or fare;
- one sentence explaining what the plot shows;
- one note about how bin size or bandwidth changes the visual story.

### 2:35-3:20 PM - Boxplots, Scatterplots, and Pairplots

Use multivariable plots to compare relationships between variables.

Notebook examples include:

- boxplots of age by ticket class;
- scatterplots of fare versus age;
- scatterplots with survival and sex as extra visual encodings;
- pairplots and scatter matrices for multiple variables.

Discussion questions:

- Did first-class passengers tend to be older or younger?
- What does a scatterplot show that a histogram cannot?
- When does adding a third or fourth visual variable make a plot more useful, and when does it make the plot too crowded?

### 3:20-3:45 PM - Pandas Plotting

Practice plotting directly from `pandas` objects.

Notebook examples include:

- line plots;
- bar and horizontal bar plots;
- histograms;
- boxplots;
- area plots;
- scatterplots.

Checkpoint:

- you can create at least two plot types using `DataFrame.plot`;
- you can explain one advantage and one limitation of quick `pandas` plotting.

### 3:45-4:35 PM - Group Exercise: Heart Disease EDA

Load `heart_disease.csv` and use plots to answer the notebook's heart disease questions.

Core tasks:

1. Use `.describe()` and missing-value checks to inspect the dataset.
2. Plot the age distribution of patients seeking cardiological exams.
3. Compare counts or percentages by sex.
4. Explore relationships among variables with pairplots, scatterplots, or a correlation heatmap.
5. Investigate whether resting blood pressure appears to increase with age.

Deliverable:

- at least three plots about the heart disease dataset;
- one short answer for each notebook question;
- one note about a limitation of the dataset or visualization.

### 4:35-5:00 PM - Checkoff and Reflection

Each group should show the TA:

1. the Titanic dataset loaded and cleaned;
2. one histogram or KDE plot;
3. one boxplot or scatterplot comparing two variables;
4. the heart disease dataset loaded from `heart_disease.csv`;
5. at least three heart disease EDA plots with brief interpretations.

## Final Questions

Answer these briefly in your group notes:

1. How do you decide whether to use a histogram, boxplot, scatterplot, or count plot?
2. Why is it important to check missing values before drawing conclusions?
3. What is one example where `seaborn` made plotting easier than raw `matplotlib`?
4. What is one heart disease variable that seems visually related to another variable?
5. What is one thing a plot can suggest but not prove?
