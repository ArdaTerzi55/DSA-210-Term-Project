# DSA210 Term Project — Alcohol Consumption and Traffic Accidents in Turkey

**Author:** Mustafa Arda Terzi
**Course:** DSA210 — Introduction to Data Science, Sabancı University, Spring 2026

---

## Motivation

Traffic accidents are one of the most preventable causes of death and injury in
Turkey. Understanding their geographic distribution — and which population-level
factors correlate with them — is a first step towards better-targeted road-safety
policy. Alcohol consumption is a well-documented contributor to traffic accidents
worldwide; this project asks whether the **regional variation** in alcohol
consumption across Turkey lines up with the regional variation in traffic
accident rates.

> **Research question:** Do Turkish provinces and sub-regions with higher
> alcohol consumption rates have significantly higher traffic accident rates
> per 100 000 population?

---

## Data Sources

The analysis combines **three official datasets**, each loaded into the
notebooks from a separate Excel file in the repository. The notebooks contain
**no hardcoded numerical data** — every value is read from an external file.

| Dataset | Source | Year | File in repo | Granularity |
| --- | --- | --- | --- | --- |
| Traffic accidents | Emniyet Genel Müdürlüğü (EGM) Trafik Hizmetleri Başkanlığı monthly bulletins | 2025 (full year) | `traffic_accidents.xlsx`, sheet `2025` | 81 provinces |
| Traffic accidents | EGM Trafik Hizmetleri Başkanlığı | 2026 Q1 | `traffic_accidents.xlsx`, sheet `2026Q1` | 81 provinces |
| Province population | TurkStat Population Projections 2023–2100 | 2025 (projection) | `Population of provinces by years.xls` | 81 provinces |
| Alcohol consumption | TurkStat Türkiye Sağlık Araştırması 2022 (microdata, ~30 000 respondents aged 15+) | 2022 | `province_alcohol_ibbs2_csv.xlsx` | 26 IBBS-2 sub-regions |

### Data enrichment

The publicly available traffic-accident counts are enriched in two ways:

1. **Population normalisation.** Raw accident counts are dominated by population
   size (Istanbul has ~16 M residents, Tunceli ~83 K). All metrics are converted
   to **accidents per 100 000 population** so provinces of different sizes can
   be compared directly.
2. **Alcohol consumption layer.** Each province is matched to its IBBS-2
   (NUTS-2) sub-region's adult alcohol-consumption rate, computed from the TUIK
   2022 health survey microdata using survey weights. This produces 26 unique
   alcohol values across the 81 provinces — a finer geographic resolution than
   the 12 IBBS-1 regions reported by TurkStat directly.

---

## Project Structure

```
DSA-210-Term-Project/
├── README.md                           ← this file
├── requirements.txt                    ← Python dependencies (versions pinned)
├── DSA210_Project_Proposal.pdf         ← initial proposal
│
├── DSA210_EDA.ipynb                    ← Phase 1: data loading + EDA
├── DSA210_Hypothesis_Testing.ipynb     ← Phase 2: six hypothesis tests
├── DSA210_Machine_Learning.ipynb       ← Phase 3: regression / classification / clustering
│
├── traffic_accidents.xlsx              ← EGM data (sheets: 2025, 2026Q1)
├── Population of provinces by years.xls← TurkStat population projections
├── province_alcohol_ibbs2_csv.xlsx     ← TUIK alcohol rates (IBBS-2 level)
│
├── data graphs/                        ← exported figures
└── raw data/                           ← original PDF / xls source files
```

---

## How to Reproduce the Analysis

### Prerequisites

- Python 3.10 or later
- pip
- Git

### Step 1 — Clone the repository

```bash
git clone https://github.com/ArdaTerzi55/DSA-210-Term-Project.git
cd DSA-210-Term-Project
```

### Step 2 — Install dependencies

```bash
pip install -r requirements.txt
```

### Step 3 — Run the three notebooks in order

Each notebook is fully self-contained — it loads the three Excel files and
runs end-to-end without depending on the others. You can run them in any
order, but the intended sequence is:

```bash
jupyter notebook DSA210_EDA.ipynb                  # ~30 s
jupyter notebook DSA210_Hypothesis_Testing.ipynb   # ~30 s
jupyter notebook DSA210_Machine_Learning.ipynb     # ~60 s
```

In each notebook, **Kernel → Restart & Run All** runs every cell from top to
bottom and produces all figures and tables. The combined runtime is roughly
two minutes on a modern laptop.

---

## Notebook 1 — `DSA210_EDA.ipynb`

| Section | Content |
| --- | --- |
| 0. Imports & Settings | Libraries, colour palette, helper functions |
| 1. Data Loading | Reads the three Excel files; produces clean per-province data |
| 2. Merging & Feature Engineering | Joins datasets, computes per-100k rates, builds 26-row sub-region aggregate |
| 3. Exploratory Data Analysis | 8 figures: distributions, top/bottom rankings, big-3 cities, sub-regional comparison, scatter plot, correlation heatmap, year-over-year trend |
| 4. EDA Summary | Plain-language interpretation |

---

## Notebook 2 — `DSA210_Hypothesis_Testing.ipynb`

Six statistical tests, all at significance level **α = 0.05**:

| # | Test | Unit of analysis | n | What it asks |
| --- | --- | --- | --- | --- |
| H1 | Welch's t-test | Province | 81 | High-alcohol provinces have higher mean accident rate |
| H2 | One-way ANOVA | Province (3 alcohol tiers) | 81 | Mean accident rate differs across alcohol tiers |
| H3 | Mann-Whitney U | **IBBS-2 sub-region** | **26** | High-alcohol sub-regions have higher accident rates |
| H4 | Kruskal-Wallis | Province (3 alcohol tiers) | 81 | Distributions differ across tiers (non-parametric) |
| H5 | Wilcoxon signed-rank | Province (paired) | 81 pairs | Accident rates changed between 2025 and projected 2026 |
| H6 | Chi-square | Province (median splits) | 81 | Alcohol-tier and accident-tier are associated |

H3 is the **most statistically honest** test: each IBBS-2 sub-region has one
alcohol value, so n=26 reflects the true number of independent observations of
the alcohol variable. Province-level tests inflate n by treating provinces as
independent even though they share their sub-region's alcohol rate.

---

## Notebook 3 — `DSA210_Machine_Learning.ipynb`

A three-part ML extension that applies methods from Recitations 7–11.

| Section | Method | Output |
| --- | --- | --- |
| Regression | Linear, kNN, Decision Tree, Random Forest with 5-fold CV | Predict `acc_per_100k` from alcohol rate + log-population |
| Classification | Logistic Regression, Random Forest with 5-fold CV | High vs low accident province (median split) |
| Unsupervised | StandardScaler → K-Means + hierarchical clustering → PCA 2D | Group the 81 provinces into 3 clusters |

The ML section also includes an explicit **IBBS-1 vs IBBS-2 comparison table**
showing how moving from 12 unique alcohol values to 26 affects model performance.

---

## Key Findings

1. **At the province level (n=81), the association is statistically significant.**
   Welch's t-test (H1), ANOVA (H2), and Kruskal-Wallis (H4) all reject the null
   at p < 0.01.

2. **At the IBBS-2 sub-region level (n=26, the more honest unit), the association
   weakens substantially.** The Mann-Whitney test (H3) fails to reject the null
   at α = 0.05 (p ≈ 0.06), and the chi-square independence test (H6) on
   median-split tiers also fails to reject (p ≈ 0.20). This is an authentic
   feature of the data: when alcohol exposure is measured at the resolution at
   which it actually varies, the link with accident rates is real but modest.

3. **Tree-based regression substantially outperforms linear regression.**
   Random Forest reaches CV R² ≈ 0.35 while linear regression stays near 0.04.
   This is a textbook signature of a non-linear interaction — the alcohol
   effect varies with population density and across regions, rather than
   following a single global slope.

4. **Three coherent clusters emerge unsupervised.** K-Means and hierarchical
   clustering agree on a low-burden eastern cluster (low accidents and low
   alcohol), a high-burden Mediterranean / Aegean coastal cluster, and a mixed
   middle group.

5. **Year-over-year (H5) shows lower projected 2026 accidents — but mostly seasonally.**
   Q1 in Turkey systematically has fewer accidents than Q2/Q3 (winter, less
   travel, less tourism). The projected decrease is a conservative lower bound,
   not evidence of a genuine safety improvement.

---

## Limitations

### Limitations

- **Geographic granularity of the alcohol data.** Alcohol consumption is
  measured at the IBBS-2 sub-region level (26 values), not per province (81).
  Even at IBBS-2 the ecological-fallacy risk is reduced but not eliminated.
- **Temporal gap.** The TUIK alcohol survey is from 2022; accident data is from
  2025–2026. Patterns of alcohol consumption may have shifted in the
  intervening years.
- **Self-reporting.** Alcohol-use surveys typically underestimate true
  consumption, especially in conservative regions.
- **Tourism distortion.** Provinces with major tourist inflows (Muğla, Antalya,
  Nevşehir) inflate accident-rate denominators because tourists drive in the
  province but are not counted in the resident population.
- **Earthquake aftermath.** Hatay (heavily affected by the 2023 earthquake)
  has unusual demographic and infrastructure conditions in this period.
- **No causal claim.** All findings document association, not causation.
  Confounders such as road quality, traffic density, vehicle fleet age,
  enforcement intensity, and weather are not in the model.
  
---

## Dependencies

See `requirements.txt` for exact version pins.

| Package | Purpose |
| --- | --- |
| pandas | Data manipulation |
| numpy | Numerical arrays |
| matplotlib | Plotting |
| seaborn | Statistical visualisations |
| scipy | Hypothesis tests |
| scikit-learn | Machine learning models |
| openpyxl | Reading `.xlsx` files |
| xlrd | Reading legacy `.xls` files |
| jupyter / notebook | Notebook environment |

---

## Use of AI Assistance

In line with the course's academic-integrity requirements, all use of AI tools
is disclosed here.

- **Tool used:** Anthropic Claude.
- **Where it was used:** guiding the project structure, assisting with code structure, debugging and clean-up of the data-loading pipeline,
- turning raw data into processed data, adding explanatory comments.

- Other works are done by hand.

---

## License and Data Attribution

This project is submitted as coursework for DSA210 at Sabancı University.
Source data is the property of the respective agencies:

- Traffic accident statistics — © Emniyet Genel Müdürlüğü, Trafik Hizmetleri Başkanlığı
- Population projections — © Türkiye İstatistik Kurumu (TurkStat / TÜİK)
- Health survey data — © Türkiye İstatistik Kurumu (TurkStat / TÜİK), Türkiye Sağlık Araştırması 2022
