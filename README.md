# Children's Screen-Time Clustering

A data-mining project that groups U.S. children with similar health, lifestyle, and environmental characteristics, and examines how screen time varies across those groups. It uses the 2024 National Survey of Children's Health (NSCH).

**Course:** ITCS444 Data Mining · **Instructor:** Aj. Charnyote Pluempitiwiriyawej

> **Project status:** preprocessing, exploratory data analysis, and the clustering analysis are complete in the notebooks, and the outputs for visualization are exported to `outputs/`. The approved primary model (complete linkage, equal weights, k = 3) gives **weakly separated clusters that change a lot with weighting and with how the neighborhood items are coded** (Sections 11 and 15). Two alternative codings of the neighborhood items were not more stable across replicate samples at k = 3, so the approved model is kept for now (Section 11). A summary for the team is in [`reports/model_summary.md`](reports/model_summary.md).

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
15. [Challenges, Failed Attempts and Lessons Learned](#15-challenges-failed-attempts-and-lessons-learned)
16. [References](#16-references)

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

*Status: implemented and run. Primary model approved on 2026-11-10: complete linkage, equal weights (W0), k = 3. Robustness checks show it is sensitive to weighting (see Section 11).*

| Step | Approach |
|---|---|
| **Distance** | **Gower distance**, implemented explicitly in NumPy. Numeric and ordinal features use the range-normalized difference, with fixed codebook ranges. Nominal and binary features use simple matching. For asymmetric binaries, a pair where both children answer "No" is excluded. Missing values are excluded pairwise. |
| **Weighting** | **Equal weights (W0)** as the baseline. Sensitivity analyses: **domain-balanced weights (W1)**, where each domain has the same total weight, and **W2**, where the three neighborhood items together carry the weight of one feature (a definition chosen by the project, not taken from a reference). |
| **Clustering** | **Hierarchical clustering** with **complete linkage** (approved). Average linkage was the planned primary but is degenerate at 10,000 children (one cluster holds 99% or more of the children in all five replicate samples). Ward, centroid, median and single linkage are not used: the first three require Euclidean distances, and single linkage chains. |
| **Sampling** | A full pairwise distance matrix for 51,375 children (about 9.8 GiB) is not feasible on the project machine. Clustering therefore runs on a **stratified sample of 10,000 children**, stratified by questionnaire form × screen-time level with random seed **42**. Five disjoint stratified replicate samples are drawn: part 0 is the primary sample and parts 1–4 are for the stability check. |
| **Assignment** | Only the 10,000 sample children are clustered hierarchically. Each remaining child is then **assigned** to the cluster with the smallest average Gower distance, and the margin to the second-closest cluster is recorded. Results always distinguish the two groups (`in_clustering_sample`). |
| **Evaluation** | Silhouette analysis for k = 2–10: the average score, scores per cluster, cluster sizes, and small clusters (under 1%, flagged for investigation rather than discarded). Final k is chosen on stability (Adjusted Rand Index across replicate samples), interpretability, and simplicity together, not on a single score. **Chosen: k = 3** (approved 2026-11-10). |
| **Sensitivity checks** | Run on part 0 for complete linkage, k = 2–4, and compared with the primary labelling using the Adjusted Rand Index against the replicate-sample stability band: W1, W2, a variant that replaces the three neighborhood items with one count (0–3), all binaries symmetric, model with `eyedoctor`, model without the sleep feature, and complete-case data |

### Key modelling decisions

| Date | Decision | Reason |
|---|---|---|
| 2026-11-10 | **Complete linkage** instead of average linkage | Average linkage is degenerate at 10,000 children in all five replicate samples. |
| 2026-11-10 | **Equal weights (W0) and k = 3** | Highest silhouette among complete-linkage k = 2-10 with no small cluster, acceptable stability, and separates an older group that k = 2 does not. |
| 2026-11-10 | **W2 definition** accepted: the three neighborhood items together carry one feature's weight | The project's own definition, used as a sensitivity run. |
| 2026-11-11 | **Neighborhood count and W2 not adopted** as primary | Not more stable than the primary model across replicate samples at k = 3. |
| 2026-11-11 | **k = 4 not adopted** | The most stable alternative (count variant, k = 4) is driven by a sex split and keeps no neighborhood information (Section 15.3). |

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
├── 03_clustering.ipynb           # Gower distance, sampling, clustering, evaluation, assignment, sensitivity, profiles, exports
├── outputs/                      # exported results (see Section 11); never contains distance matrices
├── reports/
│   └── model_summary.md          # methodology, results, limitations, guidance for using the outputs
└── README.md
```

See [Section 11](#11-outputs-and-results) for the exported files.

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
5. **Memory.** The 10,000-child clustering needs about 1.1 GiB of free RAM at its peak (the full notebook run peaked at about 1.5 GiB of process memory). Close other applications first. The notebook checks available memory before each large step. A full run of `03_clustering.ipynb` took about 35 minutes of cell time on the project machine (one run, other applications closed; expect variation).

## 10. Project Workflow

```text
children_topical.dta
      │  01_preprocessing.ipynb   select · impute · derive sleep_below_minimum · validate
      ▼
preprocessed_data.csv  (51,375 × 34)
      ├──► 02_eda.ipynb           descriptive analysis
      └──► 03_clustering.ipynb    Gower → sample (10,000) → hierarchical clustering
                                   → silhouette / stability / sensitivity → assignment
                                   → outputs/ ──► visualization & insights
```

Run the notebooks **in order: 01 → 02 → 03**. Re-run `02_eda.ipynb` after any change to `01`: its saved outputs were produced from an earlier version of the CSV, and a few imputed sleep counts have changed slightly since then. The notebooks select columns **by name**, so adding columns to the CSV does not break later notebooks.

## 11. Outputs and Results

All results come from `03_clustering.ipynb`, executed on 2026-11-10, and describe **associations in the survey sample**, not causes. They have not yet been reviewed for the final report.

### Exported files (`outputs/`)

| File | Content |
|---|---|
| `cluster_assignments.csv` | One row per child (51,375): `hhid`, `cluster` (1 = largest), `in_clustering_sample`, `assignment_method` (`hierarchical` or `assigned`), `assignment_margin`, `second_best_cluster`, `exact_tie`, ambiguity flags at 0.005 / 0.01 / 0.02, `replicate_part`. Margins are blank for the 10,000 hierarchically clustered children. |
| `evaluation_results.csv` | Silhouette, cluster sizes, small clusters for every replicate part, linkage, weighting (W0, W1) and k = 2–10 |
| `per_cluster_silhouette.csv` | Per-cluster silhouette for part 0 |
| `stability_sensitivity.csv` | The 10 replicate-sample ARIs per linkage and k (the stability band) and every sensitivity run's ARI against the primary labelling |
| `cluster_profiles.csv` | Cluster profiles, unweighted and `fwc`-weighted, with the number of children behind each statistic |
| `linkage_primary.npy`, `linkage_primary_hhid.csv` | The primary complete-linkage tree (SciPy format) and the child ID of each leaf |
| `model_config.json` | Specification, weights, linkage, k, sampling design, seed, package versions, cluster-number mapping |

Summary report: [`reports/model_summary.md`](reports/model_summary.md) (methodology, results, limitations, and guidance for using the outputs).

### Linkage choice

Average linkage failed at 10,000 children in all five replicate samples: one cluster holds 99% or more of the children at every k, and the others have 2–43 children defined by a rare answer. Its best silhouette (0.29 at k = 2, a 9,990 vs 10 split) is not meaningful. Complete linkage has no cluster under 1% for k = 2–9 in any replicate sample.

### Primary model (complete linkage, W0, k = 3)

| | Cluster 1 | Cluster 2 | Cluster 3 | All |
|---|---|---|---|---|
| Children (all 51,375, unweighted) | 33,714 (65.6%) | 14,477 (28.2%) | 3,184 (6.2%) | 51,375 |
| Of which in the 10,000 clustering sample | 6,833 | 2,594 | 573 | 10,000 |
| Share of children, weighted by `fwc` | 64.4% | 30.5% | 5.1% | 100% |
| Mean age (years) | 9.6 | 8.9 | 13.2 | 9.7 |
| Park / recreation center / library in neighborhood (%) | 96 / 69 / 96 | 37 / 6 / 12 | 46 / 10 / 22 | 77 / 47 / 68 |
| Parent-reported mental-health treatment (%) | 13 | 2 | 78 | 14 |
| Parent-reported medication for emotions, concentration or behavior (%) | 9 | 2 | 62 | 10 |
| Screen time 4+ hours (%) | 22.5 | 21.4 | 51.3 | 24.0 |
| Screen time 4+ hours, weighted (%) | 21.4 | 20.8 | 52.1 | 22.8 |

Unweighted percentages unless stated. Full profiles are in `outputs/cluster_profiles.csv`. No standard errors are given for weighted values.

- **Cluster 1 versus 2** differ mainly in neighborhood amenities; their screen-time distributions are similar.
- **Cluster 3** is a smaller, older group (72% are in the ages 12-17 questionnaire) with much higher reported mental-health treatment, medication use, and 4+ hours of screen time. Because the group is older and screen time rises with age, these differences are not evidence that treatment or medication relate to screen time on their own.
- **Quality:** mean silhouette 0.208 and cophenetic correlation 0.482, both weak. Stability across the five replicate samples: mean pairwise ARI 0.525 (range 0.302-0.805).
- **Assignment of the other 41,375 children:** no exact ties; margin to the second-closest cluster is below 0.005 for 1.9%, below 0.01 for 3.3% and below 0.02 for 5.9%. In a leave-one-out check, 89.2% of sample children return to their hierarchical cluster (cluster 1: 91.6%, cluster 2: 87.5%, **cluster 3: 67.7%**). Assigned children in cluster 3 show higher medication use (64%) than sample members of cluster 3 (53%), so the assignment rule describes that group more sharply than the tree does.

### Sensitivity (complete linkage, k = 3, full-data labellings)

The ARI of each run against the primary labelling is compared with the stability band (0.302-0.805 at k = 3):

| Run | ARI vs primary | Inside the band? | Group with the highest medication use: share of children, % medication |
|---|---|---|---|
| W1 (domain-balanced) | 0.082 | No, below | 10.7%, 65% |
| W2 (neighborhood items 1/3 each) | 0.042 | No, below | 12.0%, 61% |
| Neighborhood items replaced by one count (0-3) | 0.044 | No, below | 12.2%, 58% |
| All binaries symmetric | 0.277 | No, below | 36.6%, 17% |
| With `eyedoctor` | 0.354 | Yes | 24.3%, 19% |
| Without sleep feature | 0.481 | Yes | 14.5%, 20% |
| Complete cases only | 0.372 | Yes | 29.1%, 11% |

Reading this table:
- **The primary clusters depend on how the three correlated neighborhood items are weighted.** Down-weighting them (W2), counting them once, or using W1 gives clusters that agree with the primary clusters hardly more than chance (ARI 0.04-0.08).
- **A group with high reported mental-health treatment and medication use reappears with W1, W2 and the count variant** (about 11-12% of children, 58-65% medication, 41-47% with 4+ hours of screen time). It does **not** reappear as a distinct group in the other four runs.
- The count variant and W2 have higher silhouettes (0.218 and 0.223 at k = 3) and cophenetic correlations (0.555 and 0.532) than the primary model on part 0, but silhouettes from different distance definitions are not directly comparable. Their replicate-sample stability was therefore checked (next table).

**Replicate-sample stability of the two variants** (each clustered on all five parts, complete linkage, 10 pairwise ARIs per cell; mean, with the minimum in brackets):

| k | Primary (W0) | W2 | Neighborhood count |
|---|---|---|---|
| 2 | 0.558 (0.263) | 0.117 (-0.022) | 0.246 (-0.002) |
| 3 | 0.525 (0.302) | 0.357 (0.087) | 0.325 (0.099) |
| 4 | 0.350 (0.286) | 0.346 (0.121) | 0.539 (0.213) |

- At k = 3, the approved k, **neither variant is more stable than the primary model**, and at k = 2 both are clearly less stable. Only the count variant at k = 4 is more stable (mean 0.539, smallest cluster at least 583 children in every part); this k = 4 model has not been profiled or approved.
- The group with the highest medication use is **not consistent across parts** in either variant: it holds 8-22% of children (count variant) or 9-13% (W2), with 32-62% and 44-67% medication use respectively, while its 4+ hour screen-time share stays at 43-50%.

**Consequence:** the approved primary model is a descriptive partition that is sensitive to modelling choices, not a stable set of groups. The variants do not offer a more stable alternative at k = 3, so the primary model is kept, with this limitation stated. An exploratory k = 4 test of the count variant is described in Section 15.3, item 8.

## 12. Limitations

- **Observational, cross-sectional survey data:** results are associations, not causes. Most answers are parent reports.
- **Survey design:** clustering is **unweighted**, so it describes the survey sample, which oversamples young children and children with special health care needs. The weights (`fwc`) are kept for weighted cluster profiling, but sample-based clusters are not automatically representative of all U.S. children.
- **Sampling:** clusters are derived from a 10,000-child sample. The remaining children are *assigned* to those clusters; they did not shape them.
- **Questionnaire comparability:**
  - The original sleep codes are not comparable across forms, and `sleep_below_minimum` reduces sleep to two levels.
  - Rates of `sleep_below_minimum` change sharply at ages where the guideline changes.
  - `sleep_below_minimum` overlaps partly with age, which is also a feature.
  - `eyedoctor` is worded differently by form.
- **Imputation:** mode imputation inflates the most common category. A complete-case sensitivity run was done (Section 11).
- **Distance and weighting:** Gower results depend on feature types, weights, and how binaries are treated. Related variables (for example, the three neighborhood items) can give a domain extra weight.
- **Computation:** a full-data hierarchical clustering does not fit in memory on the project machine.
- **Weak and sensitive clusters:** silhouettes are about 0.2 or lower, and the primary k = 3 clusters change substantially when the weighting or the coding of the neighborhood items changes (Section 11). Clusters are a descriptive partition of a continuum, not natural categories.
- **Age and screen time:** the cluster with the highest screen time is also the oldest. Age is a clustering feature, so cluster differences in screen time are partly age differences.

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

**Uncommitted work (2026-11-10):** `03_clustering.ipynb` Sections 7-15 (primary clustering, evaluation, W1 sensitivity, replicate stability, decision record, assignment, further sensitivity runs, replicate stability of two variants, profiles, exports), the `outputs/` folder, and this README update.

## 15. Challenges, Failed Attempts and Lessons Learned

A running log of what did not work, what had to be changed, and what is still unresolved. It is kept so that the final report can state the limits of the analysis honestly. Entries describe the project's own work; results are associations only.

### 15.1 Variables that could not be used as planned

| Variable | Problem | Outcome |
|---|---|---|
| `physactiv` | Not asked for ages 0–5. | Dropped. |
| Height and weight | Not asked for ages 0–5. | Excluded before candidate selection. |
| `focuson` | Asked only for ages 3–5. | Excluded before candidate selection. |
| `eyecare2` | Asked only if the child saw an eye doctor. | Excluded before candidate selection. |
| `sleep_hours` (the two sleep items combined) | The T1 item is 24-hour sleep including naps, and the T2/T3 item is weeknight sleep, with different hour anchors. The combined codes are not comparable across forms and show an artificial jump at the age 5→6 boundary. | Kept for descriptive EDA only. Replaced in the model by the derived `sleep_below_minimum`. |
| `eyedoctor` | T1 asks "ever", T2/T3 ask "in the past 2 years". These cannot be reconciled for ages 2–5. | Excluded from the primary model; kept for a sensitivity run. |

### 15.2 Data and implementation difficulties

- **Sleep indicator is a compromise.** `sleep_below_minimum` reduces sleep to two levels, changes sharply where the guideline changes with age, overlaps partly with age (also a feature), and is undefined for 446 infants under 4 months (structural missingness, not nonresponse).
- **Mode imputation** fills 0.4–2.7% nonresponse per variable with the most common answer, which inflates the most common category. 4,115 children have at least one imputed value; a complete-case sensitivity run is planned.
- **No ready-made Gower package fit.** The PyPI `gower` package has no ordinal or asymmetric-binary type and would treat integer-coded nominal variables as numeric, so Gower distance was implemented directly in NumPy and validated against hand-worked examples and a pair-by-pair reference.
- **Full-data clustering does not fit in memory.** A pairwise distance matrix for 51,375 children is about 9.8 GiB, so clustering runs on a stratified sample of 10,000 children, and the remaining children are assigned afterwards. Those children did not shape the clusters.
- **SciPy's `cophenet(Z, Y)` needs about 2.3 GiB at n = 10,000.** It was replaced with a chunked implementation, checked to match SciPy exactly at n = 2,000.
- **Limited free RAM on the project machine.** The machine has 13.8 GiB, but often only 0.7–1.4 GiB was free because of other applications. A memory check (`require_memory`) stops each large step before it allocates, and the 10,000-child runs were done only after applications were closed (peak process memory 1.5 GiB).
- **Environment mismatch.** The system Python on PATH has no pandas, and the Jupyter kernel pointed to it, so `jupyter nbconvert --execute` failed. Notebooks are run with the Anaconda interpreter instead (see Section 9).
- **Notebook housekeeping.** `01_preprocessing.ipynb` declares nbformat 4.0 but contains cell ids, so `nbformat.validate` fails (harmless; not yet fixed). The saved outputs in `02_eda.ipynb` were produced before the sleep imputation fix and need a re-run.
- **Repository renamed** after the README was first written (now `ITCS444-Children-Screentime-Clustering`), so earlier links needed updating.

### 15.3 Clustering attempts that did not work as hoped

*(Results below come from `03_clustering.ipynb`, executed on 2026-11-10. Complete linkage, W0 and k = 3 were approved as the primary model at the Section 11 checkpoint; the robustness findings in items 5 and 6 may change that.)*

1. **Average linkage, the planned primary method, is degenerate.**
   - Seen first in the 2,000-child pilot and again at 10,000 in all five replicate samples.
   - One cluster holds 99% or more of the children at every k from 2 to 10; the other clusters have 2–43 children each and are defined by a rare answer (for example, mental-health care needed but not received, or vision difficulty).
   - Its best silhouette (0.29 at k = 2, a 9,990 vs 10 split) is an artefact, not evidence of structure.
2. **Complete linkage is usable but the structure is weak.**
   - Mean silhouette is at most 0.21 for any k (0.199 at k = 2, 0.208 at k = 3, 0.09–0.12 for k ≥ 5), which is below the range usually read as meaningful structure. Cophenetic correlation is 0.48, compared with 0.64 for average linkage.
   - Stability across the five replicate samples is moderate: mean pairwise ARI 0.56 at k = 2 and 0.53 at k = 3, falling to about 0.32–0.40 for larger k.
3. **Results are sensitive to weighting.**
   - Under domain-balanced weights (W1), complete linkage differs from equal weights (W0) by more than the stability band at k = 2–5.
   - The three correlated neighborhood items drive the split under W0, and sex drives it under W1. The plan therefore calls for a further W2 run.
   - A group with high rates of mental-health treatment and medication and more screen time appears under both weightings.
4. **Silhouettes from different weightings cannot be compared directly.** W0 and W1 results are compared by ARI, cluster sizes, profiles and cross-evaluation instead.
5. **The approved primary model is not robust.** Sensitivity runs with W1, W2, or one count in place of the three neighborhood items agree with the primary clusters at ARI 0.04-0.08, far below the replicate-sample band (0.30-0.81 at k = 3). A group with high reported mental-health treatment and medication use reappears in those three runs but not in the other four (all binaries symmetric, with `eyedoctor`, without sleep, complete cases).
6. **The assignment rule reproduces the smallest cluster only partly.** In a leave-one-out check, 67.7% of the 573 sample children in cluster 3 return to it (89.2% overall), and assigned cluster-3 children look more extreme than sample members.
7. **The neighborhood-item variants did not fix the instability.** Despite higher silhouettes on part 0, the W2 and count variants were not more stable across replicate samples at k = 3 (mean ARI 0.36 and 0.33 vs 0.53 for the primary model), and the high-medication group changed size and medication share from part to part.
8. **Alternative k = 4 test (exploratory, not adopted).** The neighborhood-count variant at k = 4 was the most stable cell tested (mean ARI 0.54, smallest cluster at least 583 children in every part), so it was profiled on part 0 and all children. It was run in a scratch script outside the notebook, so its numbers are not in `outputs/`. Two clusters split children by sex (1% vs 100% female; 4+ hour screen time 22.4% vs 18.6%), and two split the older high-treatment group by sleep below the minimum (2% vs 100%). Mean neighborhood amenities were about 1.9 in all four clusters, so the environment domain no longer separated anything, and agreement with the primary k = 3 labelling was ARI 0.05. Its stability comes mostly from the easy-to-reproduce sex split, so it was not adopted.

### 15.4 Open limitations and unresolved decisions

- **Weak cluster separation.** The children look more like a continuum than distinct groups. Clusters are a descriptive partition and should not be presented as natural categories.
- **Binary features dominate Gower distances.** Correlated yes/no items (the three neighborhood amenities) dominate the primary split. A single 0-3 count and W2 were tried; neither was more stable than the primary model at k = 3, so the primary model is kept. The count variant at k = 4 was more stable (mean ARI 0.54) and could be explored if the project wants a different k; that needs a new decision.
- **Cluster 3 is older.** Its higher screen time cannot be separated from age with these clusters.
- **Unweighted clustering of an oversampled survey.** Clusters describe the survey sample, not all U.S. children. Weighted standard errors would need a survey package that has not been approved.
- **Parent-reported, cross-sectional data.** No causal claims can be made. `k4q23` is parent-reported medication use, not a diagnosis.
- **W2 uses the project's own definition** (the three neighborhood items together carry one feature's weight), not one taken from a reference, and the code license is undecided.
- **Family-structure and screen-time codes** are reported by code number in the profiles; `family_r` labels were not checked against the codebook for this README.

## 16. References

### Data and documentation

- U.S. Census Bureau. *2024 National Survey of Children's Health, Topical Public Use File* [Data set]. Accessed [add retrieval date]. https://www.census.gov/programs-surveys/nsch/data/datasets/nsch2024.html
- U.S. Census Bureau. *2024 NSCH Topical Variable List* (codebook). https://www2.census.gov/programs-surveys/nsch/technical-documentation/codebook/2024-NSCH-Topical-Variable-List.pdf
- U.S. Census Bureau. *NSCH Analytic Guide.* https://www2.census.gov/programs-surveys/nsch/technical-documentation/methodology/NSCH-Analytic-Guide.pdf

The Census Bureau asks users to cite it as the source of the original data. This project's code and text are separate from the data.

### Methods

- Gower, J. C. (1971). A general coefficient of similarity and some of its properties. *Biometrics*, 27(4), 857-871. (Gower distance.)
- Kaufman, L., & Rousseeuw, P. J. (1990). *Finding Groups in Data: An Introduction to Cluster Analysis.* Wiley. (Treatment of ordinal features; interpretation of silhouette values.)
- Rousseeuw, P. J. (1987). Silhouettes: a graphical aid to the interpretation and validation of cluster analysis. *Journal of Computational and Applied Mathematics*, 20, 53-65.
- Hubert, L., & Arabie, P. (1985). Comparing partitions. *Journal of Classification*, 2(1), 193-218. (Adjusted Rand Index.)
- Paruthi, S., et al. (2016). Recommended amount of sleep for pediatric populations: a consensus statement of the American Academy of Sleep Medicine. *Journal of Clinical Sleep Medicine*, 12(6), 785-786. (Basis of the age-specific sleep minimums.)

### Software

- Virtanen, P., et al. (2020). SciPy 1.0: fundamental algorithms for scientific computing in Python. *Nature Methods*, 17, 261-272.
- Pedregosa, F., et al. (2011). Scikit-learn: machine learning in Python. *Journal of Machine Learning Research*, 12, 2825-2830.
- Versions used are listed in [Section 7](#7-tech-stack).
