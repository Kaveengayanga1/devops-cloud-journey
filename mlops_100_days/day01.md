# Day 01 - Setting up a Standardized Python ML Environment

- **Date:** May 12, 2026
- **Topic:** Python Virtual Environments and ML Package Management
- **Scenario:** xFusionCorp Industries data science team requires a standardized environment for a new ML project.

---

## 1. Task Overview

The goal was to set up a isolated Python 3 virtual environment named `ml-env` under `/root/code/`, install essential Machine Learning libraries (`numpy`, `pandas`, `scikit-learn`, `matplotlib`), and generate a `requirements.txt` file for reproducibility.

### Credentials & Environment Details

- **Server:** `controlplane`
- **User:** `root`
- **Working Directory:** `/root/code/`
- **Python Version:** 3.12.3
- **Virtual Env Name:** `ml-env`

---

## 2. Theory & Concepts

### Python Virtual Environments

A virtual environment is a self-contained directory tree that contains a Python installation for a particular version of Python, plus a number of additional packages. It prevents dependency conflicts between different projects and avoids polluting the system-wide Python installation.

### The `venv` Module

The `venv` module provides support for creating lightweight virtual environments. It is the standard way in Python 3 to manage isolated environments.

### Package Management with `pip`

`pip` is the package installer for Python. It allows you to install and manage additional libraries that are not part of the Python standard library.

### Reproducibility via `requirements.txt`

A `requirements.txt` file lists all the dependencies and their specific versions. This allows other developers or CI/CD pipelines to recreate the exact same environment using `pip install -r requirements.txt`.

---

## 3. Implementation Steps & Troubleshooting

### Terminal Output & Mistakes Made

Initially, a typo was made during the installation command:

```bash
root@controlplane ~/code ✖ python3 -m venv ml-env
root@controlplane ~/code ➜  source ml-env/bin/activate
root@controlplane ~/code via 🐍 v3.12.3 (ml-env) ➜  pip install numpy pandas scikit-learn matpotlib
```

**Mistake:** The package name `matpotlib` was misspelled (missing the 'l' in 'plot').
**Error Received:**
`ERROR: Could not find a version that satisfies the requirement matpotlib (from versions: none)`
`ERROR: No matching distribution found for matpotlib`

**Fix:** Corrected the spelling to `matplotlib`.

### Successful Execution

```bash
# Create the environment
python3 -m venv ml-env

# Activate the environment
source ml-env/bin/activate

# Install the correct packages
pip install numpy pandas scikit-learn matplotlib

# Export dependencies to requirements.txt
pip freeze > /root/code/requirements.txt

# Verify the output
cat /root/code/requirements.txt
```

---

## 4. Final `requirements.txt` Content

```text
contourpy==1.3.3
cycler==0.12.1
fonttools==4.62.1
joblib==1.5.3
kiwisolver==1.5.0
matplotlib==3.10.9
numpy==2.4.4
packaging==26.2
pandas==3.0.3
pillow==12.2.0
pyparsing==3.3.2
python-dateutil==2.9.0.post0
scikit-learn==1.8.0
scipy==1.17.1
six==1.17.0
threadpoolctl==3.6.0
```
