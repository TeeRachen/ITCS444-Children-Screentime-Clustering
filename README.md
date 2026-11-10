# Children's Screen-Time Clustering

A data-mining project that groups U.S. children with similar health, lifestyle, and environmental characteristics, and examines how screen time varies across those groups. It uses the 2024 National Survey of Children's Health (NSCH).

**Course:** ITCS444 Data Mining · **Instructor:** Aj. Charnyote Pluempitiwiriyawej

> **Project status:** preprocessing and exploratory data analysis are complete. The clustering notebook is **in progress**: the distance function, its validation, the sampling design, and a 2,000-child computational pilot are implemented. **No final clustering, evaluation, or cluster interpretation has been produced yet.**

## Contents

1. [Project Overview](#1-project-overview)
2. [Research Question and Objectives](#2-research-question-and-objectives)
3. [Dataset](#3-dataset)
4. [Data Preprocessing](#4-data-preprocessing)
5. [Exploratory Data Analysis](#5-exploratory-data-analysis)
6. [Mining Methodology](#6-mining-methodology)
7. [Tech Stack](#7-tech-stack)
8. [Repository Structure](#8-repository-structure)
9. [Installation and Setup](#9-installation-and-setup)
10. [Project Workflow](#10-project-workflow)
11. [Outputs and Results](#11-outputs-and-results)
12. [Limitations](#12-limitations)
13. [Contributors and Responsibilities](#13-contributors-and-responsibilities)
14. [Change History](#14-change-history)

---

## 1. Project Overview

Children's screen time is linked in public discussion to health, family, and neighborhood conditions, but these factors rarely act alone. This project uses **unsupervised clustering** to find groups of children who share similar combinations of characteristics, then describes the screen-time patterns within each group.

Screen time is included as one of the clustering **features**, not as a prediction target. The project describes **patterns and associations** in observational survey data and makes **no causal claims**.

## 2. Research Question and Objectives

**Research question:** *What health, lifestyle, and environmental patterns are associated with children's screen-time levels?*

**Objectives:**
- Prepare a clean, documented feature set of mixed-type survey variables (numeric, ordinal, nominal, and binary).
- Measure similarity between children with a distance suited to mixed data types (Gower distance).
- Identify groups with hierarchical clustering, and evaluate them with silhouette analysis, cluster sizes, stability, and interpretability.
- Profile each group's health, lifestyle, environmental, and screen-time characteristics.
- Report limitations, sampling caveats, and ethical considerations alongside the results.

## 3. Dataset

| Item | Description |
|---|---|
| Source | [2024 National Survey of Children's Health](https://www.census.gov/programs-surveys/nsch/data/datasets/nsch2024.html), **Topical Public Use File**, conducted by the U.S. Census Bureau |
| Population | Children aged **0–17** living in U.S. households; one sampled child per household |
| Size | **51,375 children** × 457 variables ([`datasets/children_topical.dta`](datasets/children_topical.dta), Stata format) |
| Questionnaire forms | **T1** ages 0–5 (13,841), **T2** ages 6–11 (15,574), **T3** ages 12–17 (21,960). Some questions differ by form. |
| Survey design | Complex sample with child weights (`fwc`), sampling strata (`fipsst`, `stratum`), and household ID (`hhid`). Children aged 0–5 and children with special health care needs are oversampled. |
| Documentation | [2024 NSCH Topical Variable List](https://www2.census.gov/programs-surveys/nsch/technical-documentation/codebook/2024-NSCH-Topical-Variable-List.pdf), [NSCH Analytic Guide](https://www2.census.gov/programs-surveys/nsch/technical-documentation/methodology/NSCH-Analytic-Guide.pdf) |

### Data use and citation

- **Public-use file.** The Topical PUF is a public release that the Census Bureau reviewed for disclosure avoidance. Detail that could identify respondents was removed or capped under Title 13 of the U.S. Code: for example, sub-state geography and detailed race groups, with some values suppressed (`.D`) or range-capped. More detailed data is available only through restricted access.
- **Responsible use.** This project analyzes the data in aggregate only. It makes no attempt to identify any child or household, and it does not link the file to other individual-level data. Results are reported at cluster level, never for individuals.
- **Small groups.** The NSCH Analytic Guide recommends presenting statistics only when the unweighted denominator is **30 or more**, and flagging estimates with poor reliability. Cluster profiles will follow this guidance.
- **Citation.** When reporting results, cite the U.S. Census Bureau as the source of the original data: *U.S. Census Bureau, 2024 National Survey of Children's Health, Topical Public Use File.*
- **License.** This repository has no license file. NSCH data is the U.S. Census Bureau's, and no license from this project applies to it.

## 4. Data Preprocessing

*Status: completed.* Notebook: [`01_preprocessing.ipynb`](01_preprocessing.ipynb).

1. **Variable selection.** 15 candidate variables were chosen across four domains: general, health, lifestyle, and environment.
2. **Form-specific variables:**
   - `physactiv`: dropped, because it was not asked for ages 0–5.
   - The two sleep items (`hoursleep05` for T1, `hoursleep` for T2/T3): combined into `sleep_hours`.
   - Some variables were ruled out before candidate selection: height and weight (not asked for ages 0–5), `focuson` (asked only for ages 3–5), and `eyecare2` (asked only if the child saw an eye doctor). The codebook rules and missingness rates are in the notebook.
3. **Missing values:**
   - Remaining nonresponse (0.4–2.7% per variable) was filled with the **mode**, i.e. the most common answer.
   - `sleep_hours` was imputed separately for T1 and for T2/T3, because the two forms use different hour scales.
   - Each imputed variable has a `<variable>_imputed` indicator. 47,391 children have no imputed values in the 14 clustering features.
4. **Validation.** Category codes were checked against the codebook, the notebook asserts there is no unexpected missingness, and `hhid` was confirmed as a unique, stable child identifier.
5. **Derived sleep feature: `sleep_below_minimum`.**
   - The T1 and T2/T3 sleep questions use different hour scales and measure different things (24-hour sleep including naps vs. weeknight sleep), so the combined `sleep_hours` codes are **not comparable across forms**.
   - The derived binary flag compares each child with the **minimum recommended sleep for their age** (AASM recommendations, as used by the NSCH Data Resource Center).
   - It is undefined for infants under 4 months (446 children), because no recommendation applies at that age. This missingness is structural, not nonresponse.
6. **Export.** [`datasets/preprocessed_data.csv`](datasets/preprocessed_data.csv) has 51,375 rows × 34 columns: the identifier, the 15 processed variables, survey metadata, the derived sleep flag, and the imputation indicators.

**Primary clustering specification: 14 features.** It is defined in [`03_clustering.ipynb`](03_clustering.ipynb).

| Domain | Features (Gower type) |
|---|---|
| General | `sc_age_years` (numeric), `sc_sex` (binary, symmetric), `family_r` (nominal) |
| Health | `blindness`: vision difficulty even with glasses (binary, asymmetric); `k4q22_r`: mental-health treatment (nominal); `k4q23`: medication for emotions, concentration, or behavior (binary, asymmetric) |
| Lifestyle | `screentime` (ordinal), **`sleep_below_minimum`** (binary, symmetric), `k6q71_r`: curiosity in learning (ordinal) |
| Environment | `higrade`: adult education (ordinal), `foodsit`: food affordability (ordinal), `k10q12`/`k10q13`/`k10q14`: park, recreation center, library in the neighborhood (binary, symmetric) |

Decisions about the remaining variables:
- **`sleep_hours`** is kept for descriptive EDA only.
- **`eyedoctor`** is excluded from the primary model. T1 asks whether the child has *ever* seen an eye doctor, while T2/T3 ask about the *past 2 years*. It is kept for a sensitivity analysis.

## 5. Exploratory Data Analysis

*Status: completed.* Notebook: [`02_eda.ipynb`](02_eda.ipynb).

The EDA reads the exported CSV and covers:
- the distribution of each variable
- screen time cross-tabulated against each candidate variable
- relationships between related variables (mental-health items, neighborhood amenities, education and food security)
- a feature-quality review

Its results describe associations only.

## 6. Mining Methodology

*Status: distance function, validation, sampling, and pilot implemented; full clustering and evaluation planned.*

| Step | Approach |
|---|---|
| **Distance** | **Gower distance**, implemented explicitly in NumPy. Numeric and ordinal features use the range-normalized difference, with fixed codebook ranges. Nominal and binary features use simple matching. For asymmetric binaries, a pair where both children answer "No" is excluded. Missing values are excluded pairwise. |
| **Weighting** | **Equal weights (W0)** as the baseline. **Domain-balanced weights (W1)**, where each domain has the same total weight, as a sensitivity analysis. |
| **Clustering** | **Hierarchical clustering** with **average linkage** as the primary candidate and **complete linkage** as a comparison. Ward linkage is not used, because it requires Euclidean distances. |
| **Sampling** | A full pairwise distance matrix for 51,375 children (about 9.8 GiB) is not feasible on the project machine. Clustering therefore runs on a **stratified sample of 10,000 children**, stratified by questionnaire form × screen-time level with random seed **42**. Five disjoint stratified replicate samples are drawn: part 0 is the primary sample and parts 1–4 are for the stability check. |
| **Assignment** | Only the 10,000 sample children are clustered hierarchically. Each remaining child is then **assigned** to the cluster with the smallest average Gower distance, and the margin to the second-closest cluster is recorded. Results always distinguish the two groups (`in_clustering_sample`). |
| **Evaluation** | Silhouette analysis for k = 2–10: the average score, scores per cluster, cluster sizes, and small clusters (under 1%, flagged for investigation rather than discarded). Final k is chosen on stability (Adjusted Rand Index across replicate samples), interpretability, and simplicity together, not on a single score. |
| **Sensitivity checks** | Domain-balanced weights, all binaries symmetric, model with `eyedoctor`, model without the sleep feature, and complete-case data |

## 7. Tech Stack

The versions below were verified in the project's Anaconda `base` environment.

| Tool | Version | Purpose |
|---|---|---|
| Python | 3.12.7 | Programming language |
| Jupyter notebooks (VS Code) | – | Interactive analysis |
| pandas | 2.2.2 | Data loading, cleaning, tabulation |
| NumPy | 1.26.4 | Gower distance implementation |
| SciPy | 1.13.1 | Hierarchical clustering (`linkage`, `fcluster`, `cophenet`) |
| scikit-learn | 1.5.1 | Silhouette analysis (and Adjusted Rand Index, planned) |
| matplotlib | 3.9.2 | EDA charts |
| psutil | 5.9.0 | Memory monitoring in the clustering notebook |
| Git / GitHub | – | Version control and collaboration |

## 8. Repository Structure

```text
Children-Screentime-Clustering/
├── datasets/
│   ├── children_topical.dta      # raw 2024 NSCH Topical PUF (source data)
│   └── preprocessed_data.csv     # processed dataset exported by 01 (input to 02 and 03)
├── 01_preprocessing.ipynb        # variable selection, missing values, derived sleep feature, export
├── 02_eda.ipynb                  # exploratory data analysis
├── 03_clustering.ipynb           # Gower distance, sampling, pilot (in progress)
└── README.md
```

Planned additions: `outputs/` for cluster assignments, evaluation results, and model configuration, and `reports/` for the model summary. See [Section 11](#11-outputs-and-results).

## 9. Installation and Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/TeeRachen/ITCS444-Children-Screentime-Clustering.git
   cd ITCS444-Children-Screentime-Clustering
   ```
2. **Python environment.** The notebooks were developed in the **Anaconda `base`** environment, which already includes every required package. Without Anaconda, install the libraries in [Section 7](#7-tech-stack):
   ```bash
   pip install pandas numpy scipy scikit-learn matplotlib psutil jupyter
   ```
   There is currently no pinned `requirements.txt` file.
3. **Select the kernel.** In VS Code, choose the interpreter that has these packages installed (for example, Anaconda `base`). A different system Python without pandas will fail at the first import.
4. **Data.** The raw `.dta` file and the processed CSV are included in `datasets/`. To download the raw file again, use the [NSCH 2024 data release page](https://www.census.gov/programs-surveys/nsch/data/datasets/nsch2024.html).
5. **Memory.** The planned 10,000-child clustering needs about 1.1 GiB of free RAM at its peak. Close other applications first. The notebook checks available memory before each large step.

## 10. Project Workflow

```text
children_topical.dta
      │  01_preprocessing.ipynb   select · impute · derive sleep_below_minimum · validate
      ▼
preprocessed_data.csv  (51,375 × 34)
      ├──► 02_eda.ipynb           descriptive analysis
      └──► 03_clustering.ipynb    Gower → sample (10,000) → hierarchical clustering
                                   → silhouette / stability / sensitivity → assignment
                                   → outputs/ (planned) ──► visualization & insights
```

Run the notebooks **in order: 01 → 02 → 03**. Re-run `02_eda.ipynb` after any change to `01`: its saved outputs were produced from an earlier version of the CSV, and a few imputed sleep counts have changed slightly since then. The notebooks select columns **by name**, so adding columns to the CSV does not break later notebooks.

## 11. Outputs and Results

**Existing:**
- `datasets/preprocessed_data.csv`: the processed dataset.
- Inside `03_clustering.ipynb`:
  - Gower validation: hand-worked examples and an exact match with a simple reference implementation.
  - Replicate-sample checks.
  - A pilot (n = 2,000) that measures runtime and memory. Pilot cluster results are a computational check only and are **not** interpreted.

**Planned** (not yet produced):

| File | Content |
|---|---|
| `outputs/cluster_assignments.csv` | `hhid`, cluster label, `in_clustering_sample`, assignment margin and ambiguity flags |
| `outputs/evaluation_results.csv` | Silhouette scores, cluster sizes, and stability for each linkage, weighting, and k |
| `outputs/model_config.json` | Feature specification, weights, linkage, k, sampling design, seed, package versions |
| `reports/model_summary.md` | Methodology, evaluation, interpretation of cluster labels, limitations |

Cluster profiles, visualizations, insights, and ethical reflection will be built from these outputs, without re-running the clustering.

## 12. Limitations

- **Observational, cross-sectional survey data:** results are associations, not causes. Most answers are parent reports.
- **Survey design:** clustering is **unweighted**, so it describes the survey sample, which oversamples young children and children with special health care needs. The weights (`fwc`) are kept for weighted cluster profiling, but sample-based clusters are not automatically representative of all U.S. children.
- **Sampling:** clusters are derived from a 10,000-child sample. The remaining children are *assigned* to those clusters; they did not shape them.
- **Questionnaire comparability:**
  - The original sleep codes are not comparable across forms, and `sleep_below_minimum` reduces sleep to two levels.
  - Rates of `sleep_below_minimum` change sharply at ages where the guideline changes.
  - `sleep_below_minimum` overlaps partly with age, which is also a feature.
  - `eyedoctor` is worded differently by form.
- **Imputation:** mode imputation inflates the most common category. Complete-case analysis is planned as a sensitivity check.
- **Distance and weighting:** Gower results depend on feature types, weights, and how binaries are treated. Related variables (for example, the three neighborhood items) can give a domain extra weight.
- **Computation:** a full-data hierarchical clustering does not fit in memory on the project machine.

## 13. Contributors and Responsibilities

| Role | Responsibilities |
|---|---|
| Model implementation and evaluation ([@TeeRachen](https://github.com/TeeRachen)) | Gower distance, hierarchical clustering, silhouette and stability evaluation, sensitivity analyses, exported model outputs |
| Visualization, insights, and ethical reflection ([@Chavanon-bit](https://github.com/Chavanon-bit)) | Cluster visualization and profiling, interpretation of insights, ethical reflection |

The team uses `main` for the stable, integrated project, and the feature branches `feature/model-implementation` and `feature/visualization-insights`. Changes reach `main` through pull requests.

## 14. Change History

Based on the repository's Git history:

| Commit | Date | Milestone |
|---|---|---|
| `1225b72` | 2026-11-03 | Initial commit |
| `c79b593` | 2026-11-04 | Organized files; split preprocessing and EDA into separate notebooks |
| `40bbb6d` | 2026-11-06 | Preprocessing: stable identifier (`hhid`), survey metadata, imputation indicators, sleep-scale fixes, `sleep_below_minimum` (on `feature/model-implementation`) |
| `7a30f5e` | 2026-11-06 | Sleep comparability review and `sc_age_lt4` metadata in `01_preprocessing.ipynb`; `03_clustering.ipynb` Sections 0–6 (Gower implementation and validation, replicate sampling, 2,000-child pilot); full README (on `feature/model-implementation`) |
