# DSA210 Term Project — Alcohol Consumption and Traffic Accidents in Turkey

## Overview

This project investigates whether regions in Turkey with higher alcohol
consumption rates also exhibit higher traffic accident rates per 100 000
population. The analysis combines three official datasets and applies six
statistical hypothesis tests.

**Author:** Mustafa Arda Terzi  
**Course:** DSA210 — Introduction to Data Science  
**Date:** April 2026

---

## Research Question

> Do Turkish provinces and regions with higher alcohol consumption rates have
> significantly higher traffic accident rates?

---

## Datasets

All data was manually transcribed from official PDF publications.

| Dataset | Source | Year | Granularity |
|---------|--------|------|-------------|
| Traffic accidents | Emniyet Genel Mudurlugu (EGM) — Traffic Services Presidency | 2025 (full year) | 81 provinces |
| Traffic accidents | Emniyet Genel Mudurlugu (EGM) — Traffic Services Presidency | 2026 Q1 | 81 provinces |
| Province population | TurkStat ADNKS | 2023 | 81 provinces |
| Alcohol consumption | TurkStat Turkey Saglik Arastirmasi | 2022 | 12 IBBS1 regions |

> **Important data limitation:** Alcohol data is published only at the IBBS1
> regional level (12 statistical regions). Province-level alcohol rates do not
> exist in TurkStat publications. Each province is therefore assigned the
> alcohol rate of its IBBS1 region — this introduces an ecological fallacy
> risk and is discussed in the limitations section of the notebook.

---

## Project Structure

```
DSA210-Project/
├── README.md                          # This file
├── requirements.txt                   # Python package dependencies
└── DSA210_EDA_HypothesisTesting.ipynb # Main analysis notebook
```

All data is embedded directly in the notebook as Python dictionaries
(manually transcribed from PDF sources listed above). No separate data
files are required.

---

## How to Reproduce the Analysis

### Prerequisites

- Python 3.10 or higher
- pip (Python package manager)
- Git (to clone the repository)

### Step 1 — Clone the Repository

```bash
git clone https://github.com/ArdaTerzi55/DSA-210-Term-Project
cd DSA210-Project
```

### Step 2 — Install Dependencies

```bash
pip install -r requirements.txt
```

### Step 3 — Launch Jupyter and Run the Notebook

```bash
jupyter notebook DSA210_EDA_HypothesisTesting.ipynb
```

In Jupyter, go to **Kernel → Restart & Run All** to execute every cell from
top to bottom. The full analysis takes approximately 30–60 seconds.

### Alternative — Run Without Jupyter (Script Mode)

If you prefer not to use Jupyter, you can export the notebook to a Python
script and run it directly:

```bash
jupyter nbconvert --to script DSA210_EDA_HypothesisTesting.ipynb
python DSA210_EDA_HypothesisTesting.py
```

---

## Notebook Structure

| Section | Description |
|---------|-------------|
| **0. Imports & Settings** | Libraries, colour palette, helper functions |
| **1. Data Collection** | Raw data for 2025 accidents, 2026 Q1 accidents, population, alcohol rates |
| **2. Data Merging** | Join datasets, compute per-100k rates, build regional aggregates |
| **3. EDA** | Descriptive stats, histograms, top/bottom rankings, big-3 cities, regional comparison, scatter plot, correlation heatmap, year-over-year trend |
| **4. Hypothesis Testing** | Six statistical tests (see below) |
| **5. Summary** | Consolidated results table, interpretation, limitations |

---

## Hypothesis Tests

All tests use significance level **alpha = 0.05**.

| # | Test | Data Level | n | Hypothesis |
|---|------|-----------|---|------------|
| H1 | Independent Samples T-test (Welch's) | Province | 81 | High-alcohol provinces have higher mean accident rate |
| H2 | One-Way ANOVA | Province | 81 | Mean accident rates differ across 3 alcohol tiers |
| H3 | Mann-Whitney U Test | Regional | 12 | High-alcohol regions have higher accident rates |
| H4 | Kruskal-Wallis Test | Province | 81 | Accident rate distributions differ across 3 tiers |
| H5 | Wilcoxon Signed-Rank Test | Province (paired) | 81 pairs | Accident rates changed significantly from 2025 to projected 2026 |
| H6 | Chi-Square Test of Independence | Province | 81 | Alcohol category and accident rate category are associated |

> **Note on Z-test:** Not applied because the true population variance is
> unknown. Estimating variance from sample data reduces the test to a
> t-test, making the Z-test redundant.

---

## Key Findings

- Regions with higher alcohol consumption consistently show higher
  traffic accident rates across all applicable tests.
- The association is statistically significant at alpha = 0.05 in the
  majority of tests.
- Results should not be interpreted as causal — important confounders
  include urbanisation, tourism, road infrastructure, and enforcement levels.

---

## Limitations

- Alcohol data is available only at IBBS1 level (12 regions, not 81 provinces).
- Alcohol data (2022) and accident data (2025) have a 3-year temporal gap.
- Alcohol rates are self-reported and likely underestimated.
- Tourist provinces (Mugla, Antalya) have inflated accident rates due to
  seasonal visitors not counted in the resident population.
- The 2026 projection (Q1 x 4) is a conservative lower bound because
  summer quarters historically have higher accident rates in Turkey.

---

## Dependencies

See `requirements.txt` for exact version pins.

| Package | Purpose |
|---------|---------|
| pandas | Data manipulation and DataFrames |
| numpy | Numerical arrays and calculations |
| matplotlib | Plotting engine |
| seaborn | Statistical heatmaps |
| scipy | Hypothesis tests (T-test, ANOVA, Mann-Whitney, Kruskal-Wallis, Wilcoxon, Chi-Square) |
| jupyter / notebook | Interactive notebook environment |

---

## License

This project is submitted as coursework for DSA210 at Sabanci University.
Data sourced from Emniyet Genel Mudurlugu and TurkStat — all rights reserved
by the respective agencies.
