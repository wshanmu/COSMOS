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

## Optional Tutorial: Better Terminal Setup for Linux/macOS

This tutorial is optional and is not required for Lab 1 submission. It is useful if you want a nicer terminal for the rest of the course, especially when you are navigating folders, activating Python environments, and using Git.

Goal: install [Oh My Zsh](https://ohmyz.sh/), add the [Powerlevel10k](https://github.com/romkatv/powerlevel10k) prompt theme, and enable command syntax highlighting plus autosuggestions.

### Step 0 - Check Zsh and Git

Open Terminal and run:

```bash
zsh --version
git --version
```

If `zsh` is missing on Linux, install it with your package manager. For example, on Ubuntu/Debian:

```bash
sudo apt update
sudo apt install zsh git curl
```

On macOS, Zsh is usually already installed.

Optional: make Zsh your default shell.

```bash
chsh -s "$(which zsh)"
```

Close and reopen Terminal after changing your default shell.

### Step 1 - Install Oh My Zsh

Run the official installer:

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

If the installer asks whether to switch to Zsh, choose yes.

### Step 2 - Install Powerlevel10k

Clone the theme into the Oh My Zsh custom themes folder:

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k"
```

Open `~/.zshrc` in a text editor and change the `ZSH_THEME` line to:

```bash
ZSH_THEME="powerlevel10k/powerlevel10k"
```

Restart Zsh:

```bash
exec zsh
```

If the Powerlevel10k configuration wizard does not start automatically, run:

```bash
p10k configure
```

For best icons and symbols, install the font recommended by the Powerlevel10k wizard, then select it in your terminal profile.

### Step 3 - Add Highlighting and Autosuggestions

Install the two plugins:

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions"
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting"
```

Open `~/.zshrc`, find the `plugins=(...)` line, and include these plugins. Keep `zsh-syntax-highlighting` last.

```bash
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
```

Restart Zsh:

```bash
exec zsh
```

Quick checks:

- type part of an old command and press the right arrow key to accept the autosuggestion;
- type a valid command such as `python --version` and notice the command highlighting;
- type a command that does not exist and notice that the highlighting changes.

### Troubleshooting Optional Terminal Setup

If your prompt shows boxes or question marks, install the recommended Powerlevel10k font and select it in Terminal, iTerm2, VS Code Terminal, or your Linux terminal app.

If Zsh says a plugin is missing, check that the plugin folders exist:

```bash
ls "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins"
```

If the terminal feels broken, you can temporarily disable this setup by opening `~/.zshrc`, setting:

```bash
ZSH_THEME="robbyrussell"
plugins=(git)
```

Then restart Zsh with `exec zsh`.

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
