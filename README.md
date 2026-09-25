# AI Cybersecurity — Threat & Anomaly Detection

A laboratory research project focusing on applied Machine Learning and statistical data analysis in cybersecurity. This project analyzes SSH authentication logs to identify anomalies, detect brute-force attacks, extract actionable security indicators, and prepare structured datasets for downstream anomaly detection models.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Installation & Setup with `uv`](#installation--setup-with-uv)
  - [1. Install `uv`](#1-install-uv)
  - [2. Create a Virtual Environment](#2-create-a-virtual-environment)
  - [3. Activate Environment](#3-activate-environment)
  - [4. Install Dependencies](#4-install-dependencies)
- [Running the Project](#running-the-project)
- [Pipeline Breakdown (Lab 1)](#pipeline-breakdown-lab-1)
- [Visualizations & Outputs](#visualizations--outputs)
- [Tech Stack](#tech-stack)
- [Senior Engineering Notes & Best Practices](#senior-engineering-notes--best-practices)

---

## 🛡️ Overview

Modern cybersecurity operations rely on log analysis to distinguish benign administrator activity from malicious intrusion attempts. This project processes raw and structured SSH log events (`ssh_anomaly_dataset.csv`) to:

- Quantify data quality, missingness, and feature variability.
- Perform statistical profiling and outlier detection using the Interquartile Range (IQR).
- Engineer security-focused behavioral features (e.g., failure counts per IP, targeted user diversity, temporal log spikes).
- Implement rule-based heuristic labeling to construct ground truth targets (`is_attack`).
- Generate analytical visualizations (distributions, correlations, scatter plots).
- Export sanitized, model-ready datasets for machine learning classifiers.

---

## 📁 Project Structure

```text
project_root/
├── .gitignore
├── requirements.txt
├── README.md
└── lab1/
    ├── main.py                     # Primary pipeline script (Tasks 1–11)
    ├── data/
    │   ├── ssh_anomaly_dataset.csv # Raw SSH log dataset
    │   ├── ssh_attacks.parquet     # Supplementary log data
    │   └── cleaned/
    │       └── prepared_dataset.csv # Exported model-ready dataset
    └── images/                     # Generated analytical plots
        ├── viz_1_class_dist.png    # Class distribution (Benign vs Attack)
        ├── viz_2_histogram.png     # Distribution of failed attempts per IP
        ├── viz_3_boxplot.png       # User diversity boxplot by class
        ├── viz_4_heatmap.png       # Correlation matrix heatmap
        └── viz_5_scatter.png       # Connection attempts vs failed attempts
```

---

## ⚙️ Prerequisites

- **Python**: Version `3.10` or higher
- **Package Manager**: [`uv`](https://github.com/astral-sh/uv) (ultra-fast Python package and project manager)

---

## 🚀 Installation & Setup with `uv`

We use **`uv`** for deterministic, lightning-fast virtual environment management and package resolution.

### 1. Install `uv` (if not already installed)

**Windows (PowerShell):**
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**macOS / Linux:**
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Verify installation:
```bash
uv --version
```

---

### 2. Create a Virtual Environment

Run this command in the project root (`AI_Cybersec`):

```bash
uv venv
```

*(By default, this creates a `.venv` directory in the current working directory).*

---

### 3. Activate Environment

Depending on your operating system and shell:

- **Windows (PowerShell):**
  ```powershell
  .venv\Scripts\Activate.ps1
  ```
- **Windows (Command Prompt `cmd`):**
  ```cmd
  .venv\Scripts\activate.bat
  ```
- **macOS / Linux (bash/zsh):**
  ```bash
  source .venv/bin/activate
  ```

---

### 4. Install Dependencies

Install the project dependencies using `uv pip`:

```bash
uv pip install -r requirements.txt
```

> **Tip:** Alternatively, without manually activating the virtual environment, you can run:
> ```bash
> uv run -- python ...
> ```

---

## 🏃 Running the Project

> **Important (Working Directory):**
> The script `lab1/main.py` references relative paths (e.g., `data/ssh_anomaly_dataset.csv`, `images/...`). Make sure to execute the script from inside the `lab1` directory.

### Step-by-Step Execution:

```bash
# 1. Navigate into the lab1 folder
cd lab1

# 2. Run the main processing script
python main.py
```

Or using `uv run` directly from the project root:

```bash
cd lab1
uv run python main.py
```

---

## 🔬 Pipeline Breakdown (Lab 1)

When running `lab1/main.py`, the pipeline executes the following operational stages:

1. **Data Ingestion**: Loads the raw SSH activity records.
2. **Initial Analysis**: Inspects records count, columns, data types, missing values, and unique counts.
3. **Data Quality & Hygiene**:
   - Removes duplicate entries.
   - Identifies and drops zero-variance / constant features.
   - Imputes missing numerical values via feature median.
4. **Feature Engineering**:
   - `ip_appearance_count`: Total connection frequency per IP.
   - `is_failed` & `ip_failed_attempts`: Aggregate count of failed/invalid authentication attempts per IP.
   - `ip_user_diversity`: Number of distinct usernames targeted by a single source IP (horizontal brute-force / password spraying metric).
   - `hour` & `hourly_log_volume`: Temporal activity clustering.
5. **Cyber Threat Labeling**:
   - Heuristic classification: Flags an IP as `is_attack = 1` if `ip_failed_attempts > 10` or `ip_user_diversity > 2`.
   - Computes class balance and imbalance ratio.
6. **Statistical & Outlier Detection**:
   - Computes Mean, Median, Standard Deviation, Quartiles (Q1, Q3), and IQR.
   - Flags anomalies via $1.5 \times \text{IQR}$ bounds.
7. **Correlation Analysis**: Generates Pearson correlation coefficients across numerical security indicators.
8. **Visualization Generation**: Exports 5 analytical figures to `lab1/images/`.
9. **Dataset Export**: Saves cleaned, enriched dataset to `lab1/data/cleaned/prepared_dataset.csv`.

---

## 📊 Visualizations & Outputs

The script populates `lab1/images/` with the following figures:

| File | Type | Purpose |
| :--- | :--- | :--- |
| `viz_1_class_dist.png` | Count Plot | Class balance between benign activity and identified attacks. |
| `viz_2_histogram.png` | Histogram & KDE | Distribution density of failed SSH attempts per source IP. |
| `viz_3_boxplot.png` | Box Plot | Spread of user account diversity targeted by attack vs benign traffic. |
| `viz_4_heatmap.png` | Heatmap | Pairwise correlation matrix across engineered features. |
| `viz_5_scatter.png` | Scatter Plot | Correlation between total appearances and failed attempts per IP. |

---

## 🧰 Tech Stack

- **Language**: Python 3.10+
- **Environment & Package Manager**: [uv](https://docs.astral.sh/uv/)
- **Data Manipulation**: [pandas](https://pandas.pydata.org/), [numpy](https://numpy.org/)
- **Data Visualization**: [matplotlib](https://matplotlib.org/), [seaborn](https://seaborn.pydata.org/)

---

