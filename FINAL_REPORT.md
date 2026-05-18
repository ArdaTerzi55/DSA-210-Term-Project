# DSA210 Term Project — Final Report

## Alcohol Consumption and Traffic Accidents in Turkey

**Author:** Mustafa Arda Terzi
**Course:** DSA210 — Introduction to Data Science
**Term** 2026-2027
---

## 1. Motivation

Traffic accidents are one of the most preventable causes of death and injury in
Turkey. Understanding their geographic distribution — and which population-level
factors correlate with them — is a first step toward better-targeted road-safety
policy. Alcohol consumption is a well-documented contributor to traffic accidents
worldwide.

This project investigates whether the **regional variation** in alcohol consumption
across Turkey lines up with the regional variation in traffic accident rates.

> **Research question:** Do Turkish provinces and sub-regions with higher alcohol
> consumption rates have significantly higher traffic accident rates per 100 000
> population?

---

## 2. Data Sources

The project uses three official datasets, all loaded from Excel files in the
repository.

| Dataset | Source | Year | Granularity |
|---|---|---|---|
| Traffic accidents | Emniyet Genel Müdürlüğü (EGM) — Trafik Hizmetleri Başkanlığı | 2025 full year + 2026 Q1 | 81 provinces |
| Population projections | TÜİK — Population Projections 2023–2100 | 2025 projection | 81 provinces |
| Alcohol consumption | TÜİK — Türkiye Sağlık Araştırması 2022 (microdata) | 2022 | 26 IBBS-2 sub-regions |

**Files in the repository:**
- `traffic_accidents.xlsx` — sheets `2025` and `2026Q1`, four metrics per province
  (injury/fatal accidents, material-damage accidents, fatalities, injured)
- `Population of provinces by years.xls` — official TÜİK projections
- `province_alcohol_ibbs2_csv.xlsx` — alcohol-consumption rates at the
  IBBS-2 (NUTS-2) level, computed from the TÜİK 2022 health-survey microdata
  using survey weights

**Why IBBS-2 instead of IBBS-1?** TÜİK publishes alcohol rates only at the IBBS-1
level (12 regions). For this project, the rates were re-computed from the raw
microdata at the **IBBS-2 level (26 sub-regions)** so that the geographic
resolution of the alcohol layer is roughly twice as fine. Each of the 81 provinces
is then assigned the alcohol rate of its IBBS-2 sub-region.

---

## 3. Data Analysis

The analysis is split into three Jupyter notebooks:

### 3.1 Exploratory Data Analysis — `DSA210_EDA.ipynb`

After merging the three datasets, all accident counts are converted to
**accidents per 100 000 population** so that provinces of different sizes can be
compared directly. The EDA produces eight figures:

1. Distribution of accidents per 100 000 across all provinces
2. Top-10 and bottom-10 provinces by accident rate
3. Comparison of the three big cities (Istanbul, Ankara, Izmir)
4. Sub-regional (IBBS-2) accident-rate aggregation
5. Scatter plot of alcohol rate vs accident rate
6. Correlation heatmap of all numeric features
7. Year-over-year comparison (2025 vs projected 2026)
8. Geographic ranking of provinces by alcohol-tier groups

**Key observations from EDA:**
- Istanbul has the largest **raw** accident count but a **below-average rate per
  100 000**.
- Top accident-rate provinces are concentrated in tourism/transit hubs (Muğla,
  Antalya, Osmaniye) and earthquake-affected regions (Hatay).
- Bottom accident-rate provinces sit in the Northeast and Southeast (Hakkari,
  Muş, Bitlis, Şırnak) — the same regions that show the lowest alcohol
  consumption.
- At the IBBS-2 sub-region level, the Pearson correlation between alcohol rate
  and accident rate is positive but moderate (r ≈ +0.3 to +0.5).

### 3.2 Hypothesis Testing — `DSA210_Hypothesis_Testing.ipynb`

Six statistical tests were applied at significance level **α = 0.05**:

| # | Test | Level (n) | Statistic | p-value | Decision |
|---|---|---|---|---|---|
| H1 | Welch's t-test | Province (81) | t = +3.27 | 0.0016 | **Reject H₀** |
| H2 | One-Way ANOVA | Province (81) | F = 16.20 | 1.3 × 10⁻⁶ | **Reject H₀** |
| H3 | Mann-Whitney U | Sub-region (26) | U = 116 | 0.056 | Fail to reject |
| H4 | Kruskal-Wallis | Province (81) | H = 22.32 | 1.4 × 10⁻⁵ | **Reject H₀** |
| H5 | Wilcoxon Signed-Rank | Year-over-year | W = 0 | 5.4 × 10⁻¹⁵ | **Reject H₀** (seasonal) |
| H6 | Chi-square independence | Province (81) | χ² = 1.65 | 0.199 | Fail to reject |

**Note on Z-test:** Not applied because the true population variance is unknown.
With estimated variance the Z-test reduces to a t-test, so H1 already covers
that case.

### 3.3 Machine Learning — `DSA210_Machine_Learning.ipynb`

Three families of ML methods were applied to extend the analysis from
"is there an association?" to "how well can we predict from the data?"

**Regression — predicting `acc_per_100k`:**
- Linear Regression, k-Nearest Neighbours, Decision Tree, Random Forest
- 5-fold cross-validation, metrics: R², RMSE, MAE
- **Random Forest CV R² ≈ 0.35**, Linear Regression CV R² ≈ 0.04

**Classification — predicting High vs Low accident-rate province:**
- Logistic Regression, Random Forest
- Stratified 5-fold cross-validation, metrics: accuracy, precision, recall, F1,
  ROC-AUC, confusion matrix
- **Random Forest accuracy ≈ 0.68, ROC-AUC ≈ 0.74**

**Clustering — unsupervised province grouping:**
- K-Means (k chosen by elbow + silhouette) and Hierarchical (Ward) clustering
- Features: alcohol rate, accident rate, fatality rate, population (standardised)
- PCA used for 2-D visualisation
- **Three coherent clusters emerge**, confirmed by both algorithms

All models use `random_state = 42` so results are exactly reproducible.

---

## 4. Findings

**1. The association exists at the province level.** H1, H2, and H4 all reject
the null hypothesis: provinces in higher-alcohol regions do show higher mean
accident rates. The effect is statistically very strong (p-values from
10⁻³ to 10⁻⁶).

**2. The association is weaker at the more rigorous sub-region level.**
H3 (Mann-Whitney on the 26 IBBS-2 sub-regions) gives p = 0.056 — just above the
0.05 threshold. H6 (chi-square on province categories) gives p = 0.199. This is
an honest, nuanced result: when alcohol is measured at a finer geographic scale
and provinces are not over-counted, the signal weakens.

**3. Machine learning confirms a non-linear relationship.** Random Forest
regression (CV R² ≈ 0.35) substantially out-performs Linear Regression
(CV R² ≈ 0.04). This is a classic signature of a non-linear pattern — the
alcohol-accident link is not a single global slope but interacts with
population density and region.

**4. Classification is meaningfully better than chance.** Knowing a province's
sub-regional alcohol rate plus its population class allows the model to predict
whether the province falls in the high- or low-accident half about two-thirds of
the time (ROC-AUC ≈ 0.74).

**5. Clustering reveals three province profiles:**
- A **low-burden** cluster — Northeast, Central East, and Southeast Anatolia
  provinces with low accident rates **and** low alcohol consumption.
- A **high-burden** cluster — Mediterranean and Aegean coastal provinces with
  high accident rates **and** mid-to-high alcohol consumption.
- A **mid-tier** cluster of mixed central provinces.

**6. The 2025 → 2026 year-over-year decline is seasonal, not real.** H5 rejects
strongly, but projecting only Q1 to a full year ignores the fact that Q2 and Q3
in Turkey historically have the highest accident counts.

**Overall conclusion:** the data support a **positive but moderate association**
between regional alcohol consumption and traffic accident rates in Turkey. The
association is real but smaller than naïve province-level testing suggests, and
no causal claim is made.

---

## 5. Limitations

- **Ecological fallacy.** Alcohol data is at the IBBS-2 level (26 unique values
  across 81 provinces), not truly per-province. Inferring province-level
  behaviour from sub-regional averages is a known statistical weakness.
- **Temporal gap.** The alcohol survey is from 2022; the accident data is from
  2025 / 2026 Q1.
- **Self-reporting bias.** TÜİK alcohol rates come from a self-reported survey
  and are likely under-estimated.
- **Tourism inflation.** Provinces like Muğla and Antalya have inflated accident
  rates per resident because seasonal tourists drive on their roads but are
  not counted in the resident population.
- **Earthquake aftermath.** Hatay's 2025 data reflects the unusual post-earthquake
  conditions.
- **No causal claim.** Important confounders — road quality, traffic density,
  vehicle fleet age, enforcement intensity, weather — are not in the model.

---

## 6. Future Work

- **Add vehicle and road data.** Road-network density, vehicle registrations
  per capita, and traffic-volume estimates would explain a much larger share of
  the accident-rate variance.
- **Time-series modelling.** A monthly panel from 2020 onwards would allow
  proper seasonal decomposition (the H5 weakness).
- **Better alcohol proxy.** Excise-tax revenue or alcohol-licence density at the
  province level would replace the regional self-reported rate with a more
  objective per-province measure.
- **Causal design.** A quasi-experimental approach — for example, exploiting
  policy changes that affected alcohol availability differently across regions —
  would be needed before any causal interpretation is appropriate.

---

## 7. Reproducibility

All three notebooks are self-contained and can be re-run end-to-end:

```bash
git clone https://github.com/ArdaTerzi55/DSA-210-Term-Project.git
cd DSA-210-Term-Project
pip install -r requirements.txt
jupyter notebook DSA210_EDA.ipynb
jupyter notebook DSA210_Hypothesis_Testing.ipynb
jupyter notebook DSA210_Machine_Learning.ipynb
```

Total runtime is about two minutes on a modern laptop.

**Dependencies:** pandas, numpy, matplotlib, seaborn, scipy, scikit-learn,
openpyxl, xlrd, jupyter. See `requirements.txt` for version pins.

---

## 8. Use of AI Assistance

In line with the course's academic-integrity requirements, all use of AI tools
is disclosed below.

- **Tool used:** Anthropic Claude.
- **Where it was used:** assisting with code structure, debugging, helped with README and
  report drafting, and refactoring the data-loading pipeline from hardcoded
  dictionaries to Excel-based loaders.
- **What was done by hand:** project conception, dataset selection, manual
  transcription of the EGM PDF tables into the traffic accidents Excel file,
  hypothesis design, choice of ML methods, interpretation of all results, and
  final editorial review.
- All AI-generated code was reviewed, executed, and verified before being
  committed.

---

## 9. Data Attribution

Source data is the property of the respective agencies:

- Traffic accident statistics — © Emniyet Genel Müdürlüğü, Trafik Hizmetleri Başkanlığı
- Population projections — © Türkiye İstatistik Kurumu (TÜİK)
- Health survey data — © Türkiye İstatistik Kurumu (TÜİK), Türkiye Sağlık Araştırması 2022
