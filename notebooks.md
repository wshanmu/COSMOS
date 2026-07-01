---
layout: page
title: Lab Notebooks
description: Jupyter notebooks and dataset download instructions for data science labs.
nav_order: 4
---

# Lab Notebooks

Student notebooks are hosted with the course website. Dataset files are not stored in GitHub; download them from the shared Google Drive folder and place each dataset in the same folder as the notebook that uses it.

[Download datasets from Google Drive](https://drive.google.com/drive/folders/13PVh7dG1EK2gOH9KoGR0X43AJPoya_zq?usp=sharing){: .btn .btn-primary }

## Student Notebooks

| Lab | Notebook | Dataset files to download |
| --- | --- | --- |
| Lab 1 | [Python Environment and Notebook Setup]({{ site.baseurl }}/notebooks/student/Lab_01_Python_Environment_and_Notebook_Setup/COSMOS_Lab_01_Python_Environment_and_Notebook_Setup.ipynb) | None |
| Lab 2 | [Data Analysis and Visualization]({{ site.baseurl }}/notebooks/student/Lab_02_Data_Analysis_and_Visualization/COSMOS_Lab_02_Data_Analysis_and_Visualization.ipynb) | `heart_disease.csv` |
| Lab 4 | [Linear Regression and KNN Regression]({{ site.baseurl }}/notebooks/student/Lab_04_Linear_Regression_and_KNN_Regression/COSMOS_Lab_04_Linear_Regression_and_KNN_Regression.ipynb) | `cast.csv`, `mtcars.csv` |
| Lab 7 | [Linear Regression Big Lab]({{ site.baseurl }}/notebooks/student/Lab_07_Linear_Regression_Big_Lab/COSMOS_Lab_07_Linear_Regression_Big_Lab.ipynb) | `AB_NYC_2019.csv` |
| Lab 8 | [Train/Validation/Test Split and Polynomial Regression]({{ site.baseurl }}/notebooks/student/Lab_08_Train_Validation_Test_Split_and_Polynomial_Regression/COSMOS_Lab_08_Train_Validation_Test_Split_and_Polynomial_Regression.ipynb) | `dataset_1.txt`, `league_data.txt` |
| Lab 10 | [Logistic Regression]({{ site.baseurl }}/notebooks/student/Lab_10_Logistic_Regression/COSMOS_Lab_10_Logistic_Regression.ipynb) | `nyc_airbnb.csv` |
| Lab 12 | [Classification Big Lab]({{ site.baseurl }}/notebooks/student/Lab_12_Classification_Big_Lab/COSMOS_Lab_12_Classification_Big_Lab.ipynb) | `heartdisease.csv` |

## Notes

- Keep the dataset filename unchanged after downloading it.
- Put each dataset next to the notebook before running the notebook.
- If a notebook cannot find a dataset, check the current working directory in Jupyter with:

  ```python
  import os
  os.getcwd()
  ```

- Instructor answer notebooks are kept in the private repository under `_instructor/answer-notebooks/` and are excluded from the published website.
