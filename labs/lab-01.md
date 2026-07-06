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

Use this setup if possible:

| Tool | Purpose in this course |
| --- | --- | --- |
| **Conda** | Creates an isolated Python environment so package versions do not conflict with other projects. |
| **VS Code** | Main code editor for notebooks, Python scripts, terminals, and later embedded/IoT code. |
| **Git** | Downloads and updates course code repositories, including later IoT lab code. |
| **Jupyter** | Runs `.ipynb` notebooks cell by cell for data science labs. |

Google Colab is a backup option if local installation fails, but we recommend a local setup because later labs use local datasets, saved figures, serial ports, and custom helper files.

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

## Part B: Install the Required Tools

### 1:10-1:55 PM - Install Conda, VS Code, and Git

Install these three tools first.

### 1. Install Conda

Conda is both a Python package manager and an environment manager. We will use it to create a course environment named `cosmos-ds`.

Recommended installer:

- Beginners: install [Anaconda Distribution](https://www.anaconda.com/download). It includes conda, Python, Navigator, and many common data science packages.
- Smaller install: install [Miniconda](https://www.anaconda.com/docs/getting-started/miniconda/install). It includes conda and Python, then you install only the packages you need.

After installing, open the correct terminal:

- Windows: open **Anaconda Prompt** from the Start menu.
- macOS/Linux: open **Terminal**. If `conda` is not found, close and reopen Terminal once.

Check that conda works:

```bash
conda --version
python --version
```

If `python` does not work on macOS/Linux, try:

```bash
python3 --version
```

### 2. Install Visual Studio Code

VS Code is the editor we will use to open folders, edit Python files, run notebooks, and use an integrated terminal.

Install [Visual Studio Code](https://code.visualstudio.com/).

Important macOS step: after installing VS Code, open VS Code and run:

1. Press `Command+Shift+P`.
2. Search for `Shell Command: Install 'code' command in PATH`.
3. Press `Enter`.

This lets you open a folder from Terminal with:

```bash
code .
```

### 3. Install Git

Git lets you download course code, keep it updated, and work with public repositories such as the IMU lab repository.

Install [Git](https://git-scm.com/downloads).

Platform notes:

- Windows: install Git for Windows. The default installer choices are fine. It also installs **Git Bash**, which is a useful terminal.
- macOS: installing Git may trigger an Apple Command Line Tools prompt. Accept it if prompted.
- Linux: install Git with your package manager, for example `sudo apt install git` on Ubuntu/Debian.

Check that Git works:

```bash
git --version
```

### Tool Installation Checkpoint

Show a TA that these commands work in a terminal:

```bash
conda --version
python --version
git --version
```

Use Python 3.10 or newer. Python 3.11 or 3.12 is recommended.

## Part C: Create the Course Environment

### 1:55-2:20 PM - Create an Isolated Environment

An environment keeps this course's packages separate from other projects.

Create and activate the environment:

```bash
conda create -n cosmos-ds python=3.11
conda activate cosmos-ds
```

Install the data science and notebook packages:

```bash
python -m pip install --upgrade pip
python -m pip install numpy pandas matplotlib seaborn scipy scikit-learn jupyterlab notebook ipykernel
```

Register the environment as a Jupyter kernel:

```bash
python -m ipykernel install --user --name cosmos-ds --display-name "Python (COSMOS DS)"
```

Every time you start work for this course, activate the environment first:

```bash
conda activate cosmos-ds
```

Checkpoint:

```bash
python -c "import numpy, pandas, matplotlib, sklearn; print('DS environment ready')"
```

## Part D: Configure VS Code for Python and Jupyter

### 2:20-2:50 PM - Install Extensions and Select the Environment

Open VS Code and install the required extensions:

1. Open the Extensions view:
   - macOS: `Command+Shift+X`
   - Windows/Linux: `Ctrl+Shift+X`
2. Search for **Python**.
3. Install **Python** by Microsoft.
4. Search for **Jupyter**.
5. Install **Jupyter** by Microsoft.
6. If VS Code asks to reload, reload the window.

Open the course folder:

1. In VS Code, choose **File > Open Folder**.
2. Select your `COSMOS-Cluster10` folder.
3. Choose **Terminal > New Terminal**.
4. In the VS Code terminal, run:

```bash
conda activate cosmos-ds
```

Select the Python interpreter:

1. Open the Command Palette:
   - macOS: `Command+Shift+P`
   - Windows/Linux: `Ctrl+Shift+P`
2. Search for `Python: Select Interpreter`.
3. Choose the interpreter that says `cosmos-ds`.
4. If there are multiple choices, choose the one whose path includes `envs/cosmos-ds`.

Open a notebook and select the kernel:

1. Open the Lab 1 `.ipynb` file.
2. Click **Select Kernel** in the upper-right corner of the notebook.
3. Select **Python (COSMOS DS)**.
4. Run the first cell.

If the notebook cannot find the kernel, restart VS Code and run this again in the VS Code terminal:

```bash
conda activate cosmos-ds
python -m ipykernel install --user --name cosmos-ds --display-name "Python (COSMOS DS)"
```

### Useful VS Code Shortcuts

You do not need to memorize all of these today, but using them will make labs much faster.

| Action | macOS | Windows/Linux |
| --- | --- | --- |
| Command Palette | `Command+Shift+P` | `Ctrl+Shift+P` |
| Quick open file | `Command+P` | `Ctrl+P` |
| Open/close terminal | `` Control+` `` | `` Ctrl+` `` |
| Find in current file | `Command+F` | `Ctrl+F` |
| Save file | `Command+S` | `Ctrl+S` |
| Move cursor by word | `Option+Left/Right` | `Ctrl+Left/Right` |
| Select by word | `Option+Shift+Left/Right` | `Ctrl+Shift+Left/Right` |
| Move to line start/end | `Command+Left/Right` | `Home` / `End` |
| Comment/uncomment line | `Command+/` | `Ctrl+/` |
| Move current line | `Option+Up/Down` | `Alt+Up/Down` |
| Run selected notebook cell | `Control+Enter` | `Ctrl+Enter` |
| Run cell and move to next | `Shift+Enter` | `Shift+Enter` |
| Run cell and insert below | `Option+Enter` | `Alt+Enter` |

Notebook tip: if a keyboard shortcut seems to edit text instead of controlling the notebook, click outside the cell text area or press `Esc` first.

## Part E: Run the Lab 1 Notebook Skeleton

### 2:50-3:20 PM - Open the Provided Notebook

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

## Optional Setup for Later IoT Labs

If you finish early, install the **ESP-IDF** extension in VS Code. We will use it later for ESP32-based WiFi and IMU labs.

1. Open the Extensions view:
   - macOS: `Command+Shift+X`
   - Windows/Linux: `Ctrl+Shift+X`
2. Search for **ESP-IDF**.
3. Install the **Espressif IDF** extension.
4. Open the Command Palette:
   - macOS: `Command+Shift+P`
   - Windows/Linux: `Ctrl+Shift+P`
5. Search for `ESP-IDF: Open ESP-IDF Installation Manager`.
6. Start the installer if you have time. If it takes too long, stop here and finish it during the WiFi or IMU lab with a TA.

Official reference: [ESP-IDF VS Code extension installation guide](https://docs.espressif.com/projects/vscode-esp-idf-extension/en/latest/installation.html).
