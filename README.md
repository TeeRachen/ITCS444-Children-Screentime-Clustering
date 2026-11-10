# Children's Screen-Time Clustering

A data-mining project that groups U.S. children with similar health, lifestyle, and environmental characteristics, and examines how screen time varies across those groups. It uses the 2024 National Survey of Children's Health (NSCH).

**Course:** ITCS444 Data Mining · **Instructor:** Aj. Charnyote Pluempitiwiriyawej

> **Status (2026-11-11):** preprocessing, exploratory analysis and clustering are done, the results are exported to `outputs/`, and a model summary is in [`reports/`](reports/model_summary.md). The primary model (complete linkage, equal weights, k = 3) finds **weakly separated clusters that change with the weighting**. Read them as a description of this survey sample, not as natural groups. Visualization and ethical reflection are the teammate's part.

## Contents

1. [Overview](#1-overview)
2. [Dataset](#2-dataset)
3. [Method](#3-method)
4. [Results](#4-results)
5. [Outputs](#5-outputs)
6. [Run it](#6-run-it)
7. [Limitations](#7-limitations)
8. [Team and Change History](#8-team-and-change-history)
9. [References](#9-references)

---

## 1. Overview

Children's screen time is linked in public discussion to health, family, and neighborhood conditions, but these factors rarely act alone. This project uses **unsupervised clustering** to find groups of children who share similar combinations of characteristics, then describes the screen-time patterns within each group.

**Research question:** *What health, lifestyle, and environmental patterns are associated with children's screen-time levels?*

Screen time is a clustering **feature**, not a prediction target. The project describes **associations** in observational survey data and makes **no causal claims**.

## 2. Dataset

| Item | Description |
|---|---|
| Source | [2024 National Survey of Children's Health](https://www.census.gov/programs-surveys/nsch/data/datasets/nsch2024.html), **Topical Public Use File**, U.S. Census Bureau |
| Population | Children aged **0–17** in U.S. households, one sampled child per household |
| Size | **51,375 children** × 457 variables ([`datasets/children_topical.dta`](datasets/children_topical.dta)) |
| Forms | **T1** ages 0–5 (13,841), **T2** ages 6–11 (15,574), **T3** ages 12–17 (21,960). Some questions differ by form. |
| Survey design | Child weights (`fwc`), strata (`fipsst`, `stratum`), household ID (`hhid`). Ages 0–5 and children with special health care needs are oversampled. |

**Data use.**
- The file is a public release that the Census Bureau reviewed for disclosure avoidance. This project analyzes it in aggregate only and makes no attempt to identify any child or household.
- Following the NSCH Analytic Guide, statistics are shown only when the unweighted denominator is **30 or more**.
- Cite the U.S. Census Bureau as the source of the original data (see [References](#9-references)). The repository has **no license file**, and nothing here applies to the NSCH data.

## 3. Method

**Preprocessing** ([`01_preprocessing.ipynb`](01_preprocessing.ipynb)):
- 15 candidate variables across four domains; variables not asked for all ages were excluded (details in [`docs/challenges.md`](docs/challenges.md)).
- Nonresponse (0.4–2.7% per variable) is filled with the **mode**, with an `<variable>_imputed` flag for each. 47,391 children have no imputed value in the 14 clustering features.
- The two sleep questions are not comparable across forms, so the model uses a derived flag, **`sleep_below_minimum`**, that compares each child with the recommended minimum for their age. It is undefined (structurally missing) for 446 infants under 4 months.
- Export: [`datasets/preprocessed_data.csv`](datasets/preprocessed_data.csv), 51,375 rows × 34 columns. Exploratory analysis is in [`02_eda.ipynb`](02_eda.ipynb).

**The 14 clustering features** ([`03_clustering.ipynb`](03_clustering.ipynb)):

| Domain | Features (Gower type) |
|---|---|
| General | age (numeric), sex (binary), family structure (nominal) |
| Health | vision difficulty even with glasses (asymmetric binary), mental-health treatment (nominal), medication for emotions, concentration or behavior (asymmetric binary) |
| Lifestyle | screen time (ordinal), sleep below age minimum (binary), curiosity in learning (ordinal) |
| Environment | highest adult education (ordinal), food affordability (ordinal), park / recreation center / library in the neighborhood (3 binaries) |

Medication use is parent-reported, not a diagnosis. `eyedoctor` is excluded because its wording differs by form.

**Clustering approach:**

| Step | Approach |
|---|---|
| Distance | **Gower distance**, implemented in NumPy and validated against hand-worked examples and a pair-by-pair reference. |
| Weights | Equal weights (W0). Sensitivity runs use domain-balanced weights (W1) and W2, where the three neighborhood items together carry one feature's weight (the project's own definition). |
| Linkage | **Complete linkage**. Average linkage was the plan but is degenerate at 10,000 children. |
| Sampling | A full distance matrix for 51,375 children (about 9.8 GiB) does not fit in memory, so **10,000 children** (stratified by form × screen time, seed 42) are clustered. Five disjoint replicate samples are drawn for stability checks. |
| Assignment | The other 41,375 children join the cluster with the smallest average distance. The margin to the runner-up is recorded. |
| Evaluation | Silhouette (k = 2–10), cluster sizes, stability (Adjusted Rand Index across the five samples), and sensitivity runs. |

**Key decisions:**

| Date | Decision | Reason |
|---|---|---|
| 2026-11-10 | Complete linkage | Average linkage puts 99% or more of the children in one cluster in all five samples. |
| 2026-11-10 | W0 and k = 3 | Best silhouette among k = 2–10, no small cluster, acceptable stability, separates an older group. |
| 2026-11-11 | Neighborhood count and W2 not adopted | Not more stable than the primary model at k = 3. |
| 2026-11-11 | k = 4 not adopted | The most stable alternative is driven by a sex split and keeps no neighborhood information. |

## 4. Results

Primary model (complete linkage, W0, k = 3). Full tables are in [`reports/model_summary.md`](reports/model_summary.md).

| | Cluster 1 | Cluster 2 | Cluster 3 |
|---|---|---|---|
| Children (of 51,375) | 33,714 (65.6%) | 14,477 (28.2%) | 3,184 (6.2%) |
| Mean age (years) | 9.6 | 8.9 | 13.2 |
| Park / recreation center / library nearby (%) | 96 / 69 / 96 | 37 / 6 / 12 | 46 / 10 / 22 |
| Parent-reported mental-health treatment (%) | 13 | 2 | 78 |
| Screen time 4+ hours (%) | 22.5 | 21.4 | 51.3 |

- **Clusters 1 and 2** differ mainly in neighborhood amenities. Their screen-time levels are similar.
- **Cluster 3** is older, with much higher reported treatment, medication use and screen time. Screen time rises with age, so these differences do not show that treatment or medication relate to screen time on their own.
- **The structure is weak.** Mean silhouette is 0.208, and the mean ARI between replicate samples is 0.525 (range 0.302–0.805).
- **The clusters are sensitive to modelling choices.** With W1, W2, or one count in place of the three neighborhood items, the clusters agree with the primary ones at ARI 0.04–0.08, far below that range. The two neighborhood variants were not more stable at k = 3, so the primary model is kept.
- **Assignment:** 5.9% of the assigned children have a margin below 0.02. In a leave-one-out check, 89.2% of sample children return to their cluster, but only 67.7% in cluster 3.

## 5. Outputs

Written to `outputs/` by the clustering notebook. Distance matrices are never saved.

| File | Content |
|---|---|
| `cluster_assignments.csv` | One row per child: `hhid`, `cluster` (1 = largest), `in_clustering_sample`, `assignment_method`, margin to the runner-up, ambiguity flags, `replicate_part` |
| `cluster_profiles.csv` | Profiles, unweighted and `fwc`-weighted, with the number of children behind each statistic |
| `evaluation_results.csv`, `per_cluster_silhouette.csv` | Silhouette, sizes and small clusters by part, linkage, weighting and k |
| `stability_sensitivity.csv` | Replicate-sample ARIs and each sensitivity run's ARI against the primary labelling |
| `linkage_primary.npy`, `linkage_primary_hhid.csv` | The primary tree (SciPy format) and the child ID of each leaf |
| `model_config.json` | Specification, weights, linkage, k, sampling design, seed, package versions |

## 6. Run it

```text
Children-Screentime-Clustering/
├── datasets/             # raw .dta and preprocessed_data.csv
├── 01_preprocessing.ipynb
├── 02_eda.ipynb
├── 03_clustering.ipynb
├── outputs/              # exported results
├── reports/model_summary.md
├── docs/challenges.md    # what did not work, with details
└── README.md
```

1. **Clone:** `git clone https://github.com/TeeRachen/ITCS444-Children-Screentime-Clustering.git`
2. **Environment:** the notebooks were developed in Anaconda `base`: Python 3.12.7, pandas 2.2.2, NumPy 1.26.4, SciPy 1.13.1, scikit-learn 1.5.1, matplotlib 3.9.2, psutil 5.9.0. There is no `requirements.txt`. In VS Code, select an interpreter that has these packages.
3. **Run in order:** 01 → 02 → 03. Re-run `02_eda.ipynb` after any change to `01`; its saved outputs predate a small sleep-imputation fix.
4. **Resources:** the clustering needs about 1.1 GiB of free RAM at its peak (about 1.5 GiB for the whole notebook). A full run of `03_clustering.ipynb` took about 35 minutes of cell time on the project machine. The notebook checks memory before each large step, so close other applications first.

## 7. Limitations

- **Associations only.** The data are observational, cross-sectional and mostly parent-reported.
- **Unweighted clustering of an oversampled survey.** The clusters describe the survey sample, not all U.S. children. Weighted profiles have no standard errors.
- **Weak, sensitive clusters.** Silhouettes are about 0.2, and the clusters change with the weighting and the coding of the neighborhood items. They are a descriptive partition of a continuum.
- **Age and screen time overlap.** The highest-screen-time cluster is also the oldest, and age is a clustering feature.
- **Sampling and imputation.** Clusters are built from 10,000 children and the rest are assigned to them. Mode imputation inflates the most common answer.
- **Sleep and questionnaire comparability.** `sleep_below_minimum` is a two-level summary that overlaps with age.

The full log of failed attempts, difficulties and open decisions is in [`docs/challenges.md`](docs/challenges.md).

## 8. Team and Change History

| Role | Responsibilities |
|---|---|
| Model implementation and evaluation ([@TeeRachen](https://github.com/TeeRachen)) | Gower distance, hierarchical clustering, evaluation, sensitivity analyses, exported outputs |
| Visualization, insights, and ethical reflection ([@Chavanon-bit](https://github.com/Chavanon-bit)) | Cluster visualization and profiling, interpretation, ethical reflection |

Changes reach `main` through pull requests.

| Commit | Date | Milestone |
|---|---|---|
| `1225b72` | 2026-11-03 | Initial commit |
| `c79b593` | 2026-11-04 | Organized files; split preprocessing and EDA into separate notebooks |
| `40bbb6d` | 2026-11-06 | Preprocessing: identifier, survey metadata, imputation flags, sleep fixes, `sleep_below_minimum` |
| `7a30f5e` | 2026-11-06 | `03_clustering.ipynb` Sections 0–6 (Gower, sampling, pilot), sleep review, first full README |
| `aa7d7ba` | 2026-11-11 | Clustering, evaluation, sensitivity checks, assignment, profiles, exports (`outputs/`) |
| `f66377d` | 2026-11-11 | Model summary and README with results |

## 9. References

**Data and documentation**
- U.S. Census Bureau. *2024 National Survey of Children's Health, Topical Public Use File* [Data set]. Accessed 21 September 2026. https://www.census.gov/programs-surveys/nsch/data/datasets/nsch2024.html
- U.S. Census Bureau. *2024 NSCH Topical Variable List* (codebook). https://www2.census.gov/programs-surveys/nsch/technical-documentation/codebook/2024-NSCH-Topical-Variable-List.pdf
- U.S. Census Bureau. *NSCH Analytic Guide.* https://www2.census.gov/programs-surveys/nsch/technical-documentation/methodology/NSCH-Analytic-Guide.pdf

**Methods**
- Gower, J. C. (1971). A general coefficient of similarity and some of its properties. *Biometrics*, 27(4), 857-871.
- Kaufman, L., & Rousseeuw, P. J. (1990). *Finding Groups in Data: An Introduction to Cluster Analysis.* Wiley.
- Rousseeuw, P. J. (1987). Silhouettes: a graphical aid to the interpretation and validation of cluster analysis. *Journal of Computational and Applied Mathematics*, 20, 53-65.
- Hubert, L., & Arabie, P. (1985). Comparing partitions. *Journal of Classification*, 2(1), 193-218.
- Paruthi, S., et al. (2016). Recommended amount of sleep for pediatric populations: a consensus statement of the American Academy of Sleep Medicine. *Journal of Clinical Sleep Medicine*, 12(6), 785-786. (Basis of the age-specific sleep minimums.)

**Software**
- Virtanen, P., et al. (2020). SciPy 1.0: fundamental algorithms for scientific computing in Python. *Nature Methods*, 17, 261-272.
- Pedregosa, F., et al. (2011). Scikit-learn: machine learning in Python. *Journal of Machine Learning Research*, 12, 2825-2830.
