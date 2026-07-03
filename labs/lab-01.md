---
layout: page
title: "Lab 1: Python Environment and Notebook Setup"
description: DS lab for preparing Python, an IDE, Jupyter, and the course notebook skeleton.
parent: Labs
nav_order: 1
---

# Lab 1: Python Environment and Notebook Setup **DS**{: .label .label-ds }

Date: July 6  
Time: 1:00-5:00 PM  
TA: Jianhao Huang / Shanmu Wang  
Goal: Prepare a working Python environment for all data science labs.

## Why This Lab Matters

Most data science labs in this course will be distributed as Jupyter notebook skeletons. Before we start modeling and visualization, everyone needs a reliable setup that can:

- open `.ipynb` notebooks;
- run Python code cells;
- import common data science packages;
- save outputs and figures;
- keep course files organized.

By the end of this lab, you should have one working setup on your laptop or lab computer and be able to run the provided notebook skeleton from start to finish.

## Recommended Setup

Use one of these setups:

| Option | Recommended For | Notes |
| --- | --- | --- |
| VS Code + Jupyter extension | Most students | Good editor, file browser, terminal, and notebook support in one app. |
| JupyterLab in browser | Students who prefer browser notebooks | Simple and close to classic notebook workflows. |
| Google Colab | Backup only | Useful if local installation fails, but local files and packages are less convenient. |

We recommend a local Python environment because later labs may use local datasets, saved figures, and custom helper files.

## Part A: Organize Your Course Folder

### 1:00-1:10 PM - Create a Course Workspace

Create one folder for the course, for example:

```text
COSMOS-Cluster10/
  labs/
  data/
  outputs/
  notebooks/
```

Suggested rule: one subfolder per lab.

```text
COSMOS-Cluster10/
  labs/
    lab01-python-setup/
    lab02-data-visualization/
    lab04-regression-knn/
```

Do not put course work only in `Downloads`. It is easy to lose files there.

## Part B: Install Python

### 1:10-1:35 PM - Choose One Python Distribution

Use one of the following.

### Option 1: Miniforge or Miniconda

This is recommended if you already know conda or want a clean environment manager.

After installing, open a terminal and check:

```bash
python --version
conda --version
```

If `python` does not work on macOS, try:

```bash
python3 --version
```

### Option 2: Python.org Installer

Install Python from the official Python installer. Then check:

```bash
python --version
python -m pip --version
```

On macOS, you may need:

```bash
python3 --version
python3 -m pip --version
```

### Required Python Version

Use Python 3.10 or newer. Python 3.11 or 3.12 is recommended. If your Python version is older than 3.10, ask a TA before continuing.

## Part C: Create the Course Environment

### 1:35-2:00 PM - Create an Isolated Environment

An environment keeps this course's packages separate from other projects.

### If Using Conda

```bash
conda create -n cosmos-ds python=3.11
conda activate cosmos-ds
```

Install packages:

```bash
python -m pip install numpy pandas matplotlib seaborn scipy scikit-learn jupyterlab notebook ipykernel
```

### If Using venv

Create the environment inside your course folder.

macOS/Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install numpy pandas matplotlib seaborn scipy scikit-learn jupyterlab notebook ipykernel
```

Windows PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install numpy pandas matplotlib seaborn scipy scikit-learn jupyterlab notebook ipykernel
```

Checkpoint:

```bash
python -c "import numpy, pandas, matplotlib, sklearn; print('DS environment ready')"
```

## Part D: Install and Configure an IDE

### 2:00-2:25 PM - VS Code Setup

If using VS Code:

1. Install VS Code.
2. Install these extensions:
   - Python
   - Jupyter
3. Open your `COSMOS-Cluster10` folder in VS Code.
4. Open a terminal inside VS Code.
5. Activate your environment.
6. Select the Python interpreter:
   - press `Command+Shift+P` on macOS or `Ctrl+Shift+P` on Windows;
   - search for `Python: Select Interpreter`;
   - choose `cosmos-ds` or `.venv`.

### 2:25-2:35 PM - Register a Jupyter Kernel

Run:

```bash
python -m ipykernel install --user --name cosmos-ds --display-name "Python (COSMOS DS)"
```

When opening a notebook, select the kernel named:

```text
Python (COSMOS DS)
```

## Part E: Run the Lab 1 Notebook Skeleton

### 2:35-3:20 PM - Open the Provided Notebook

Download or copy the provided Lab 1 notebook skeleton into:

```text
COSMOS-Cluster10/labs/lab01-python-setup/
```

Expected file name:

```text
lab01_python_setup.ipynb
```

Open it in VS Code or JupyterLab.

If using JupyterLab, start it from the activated environment:

```bash
jupyter lab
```

Then open the notebook from the file browser.

### Notebook Check Cells

Your notebook should include or run equivalent cells:

```python
import sys
print(sys.version)
print(sys.executable)
```

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import sklearn

print("numpy:", np.__version__)
print("pandas:", pd.__version__)
print("scikit-learn:", sklearn.__version__)
```

```python
x = np.linspace(0, 2 * np.pi, 200)
y = np.sin(x)

plt.plot(x, y)
plt.xlabel("x")
plt.ylabel("sin(x)")
plt.title("Environment test plot")
plt.show()
```

Checkpoint: show the TA your notebook with all cells executed and a visible plot.

## Part F: Python Warm-Up

### 3:20-4:15 PM - NumPy and Dictionaries

Complete the warm-up cells in the skeleton notebook.

Topics:

- lists vs NumPy arrays;
- array indexing and slicing;
- basic array statistics: mean, standard deviation, min, max;
- dictionaries for storing structured records;
- converting a list of dictionaries into a `pandas.DataFrame`.

Example data structure:

```python
records = [
    {"student": "A", "sensor": "imu", "samples": 1200, "label": "walking"},
    {"student": "B", "sensor": "uwb", "samples": 800, "label": "los"},
    {"student": "C", "sensor": "mmwave", "samples": 500, "label": "gesture"},
]
```

Required tasks:

1. Create a DataFrame from the records.
2. Add one new record.
3. Compute total samples by sensor type.
4. Make one bar chart.
5. Save the figure into the `outputs/` folder.

## Part G: Debugging Practice

### 4:15-4:40 PM - Common Errors

Work through at least three intentionally broken cells in the skeleton notebook.

Examples:

- missing import;
- wrong variable name;
- wrong file path;
- package not installed;
- kernel using the wrong Python environment.

For each error, write:

1. What error message did Python show?
2. What caused the error?
3. How did you fix it?

## Part H: Reproducibility Check

### 4:40-4:55 PM - Restart and Run All

In the notebook menu:

1. Restart the kernel.
2. Run all cells from top to bottom.
3. Confirm there are no hidden dependencies on old variables.
4. Save the notebook.

This is important. A notebook is not complete until it can run from a clean kernel.

## Submit

Submit:

1. Completed `lab01_python_setup.ipynb`.
2. One screenshot showing the selected kernel.
3. One screenshot or output cell showing package versions.
4. One generated plot saved in `outputs/`.
5. Short answers to the debugging questions.

## Troubleshooting

### `ModuleNotFoundError`

The package is missing from the current environment.

```bash
python -m pip install package_name
```

Then restart the notebook kernel.

### Notebook Uses the Wrong Python

Run this in a notebook cell:

```python
import sys
print(sys.executable)
```

If it does not point to `cosmos-ds` or `.venv`, select the correct kernel.

### `jupyter: command not found`

Activate the environment and install Jupyter:

```bash
python -m pip install jupyterlab notebook
```

Then run:

```bash
jupyter lab
```

### File Path Problems

Use paths relative to the notebook location when possible.

Recommended:

```python
from pathlib import Path

project_dir = Path.cwd()
data_dir = project_dir / "data"
output_dir = project_dir / "outputs"
output_dir.mkdir(exist_ok=True)
```

Avoid hard-coded paths like:

```text
/Users/myname/Downloads/random_folder/data.csv
```

## What to Conclude

A strong Lab 1 setup means:

- you can open and run notebooks;
- the notebook uses the intended Python environment;
- common packages import successfully;
- plots display and save correctly;
- your files are organized for the rest of the course.

If your environment still does not work by the end of lab, ask a TA to help you switch to the backup option before the next data science lab.
