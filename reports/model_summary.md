# Model Summary

The primary model is Gower distance, complete linkage, equal weights (W0), k = 3. It finds two groups that differ by neighborhood amenities and one older group with high reported treatment and screen time. The structure is weak and sensitive to weighting.

**Source of every number:** `03_clustering.ipynb` (run 2026-11-10 and 2026-11-11) and `outputs/`. **Data:** 2024 NSCH Topical Public Use File, U.S. Census Bureau.

> The clusters describe **associations in a survey sample**. They are not causes, diagnoses or natural categories of children. Use them to describe group tendencies, never to label an individual child.

## Contents

1. [What question and approach?](#1-what-question-and-approach)
2. [Which data and features?](#2-which-data-and-features)
3. [How does the method work?](#3-how-does-the-method-work)
4. [Why complete linkage and k = 3?](#4-why-complete-linkage-and-k--3)
5. [What do the three clusters look like?](#5-what-do-the-three-clusters-look-like)
6. [How good and how robust is the model?](#6-how-good-and-how-robust-is-the-model)
7. [What are the limitations?](#7-what-are-the-limitations)
8. [How to use the outputs](#8-how-to-use-the-outputs)
9. [References](#9-references)

## 1. What question and approach?

- **Research question:** what health, lifestyle and environmental patterns are associated with children's screen-time levels?
- Children with similar characteristics are grouped by hierarchical clustering. Screen time is a clustering **feature**, not a predicted outcome.

## 2. Which data and features?

- 51,375 children aged 0–17, one per household. Forms: T1 ages 0–5, T2 6–11, T3 12–17.
- The survey oversamples ages 0–5 and children with special health care needs. The clustering is **unweighted**, so it describes the survey sample, not all U.S. children.
- Mode imputation fills 0.4–2.7% nonresponse per variable; 47,391 children have no imputed value in the 14 features.

| Domain | Features |
|---|---|
| General | Age, sex, family structure |
| Health | Vision difficulty even with glasses; mental-health treatment in the past 12 months; medication for emotions, concentration or behavior in the past 12 months (parent-reported, **not a diagnosis**) |
| Lifestyle | Screen time (5 weekday levels), sleep below the age-specific minimum, curiosity in learning |
| Environment | Highest adult education, food affordability, park or playground / recreation center / library in the neighborhood |

**Not used:** `eyedoctor` (worded differently by form), the original sleep-hours codes (not comparable across forms), height, weight, `focuson`, `eyecare2`, `physactiv` (not asked for all ages).

## 3. How does the method work?

| Step | Choice |
|---|---|
| Distance | **Gower distance** in NumPy, validated against hand-worked examples and a pair-by-pair reference (exact match). Ordinal and numeric: range-normalized difference. Nominal and binary: simple matching. Asymmetric binaries (vision difficulty, medication): two "No" answers do not count as similar. |
| Weights | **Equal weights (W0)** |
| Clustering | **Hierarchical, complete linkage**, k = 3 |
| Sample | **10,000 children**, stratified by form × screen-time level, seed 42. The other **41,375 are assigned** to the cluster with the smallest average distance to its sample members. |
| Evaluation | Mean and per-cluster silhouette (k = 2–10), cluster sizes, ARI across five disjoint stratified replicate samples, sensitivity runs |

A full 51,375 × 51,375 distance matrix (about 9.8 GiB) does not fit in memory, hence the sample-and-assign design.

## 4. Why complete linkage and k = 3?

**Average linkage was rejected.** At 10,000 children it put 99% or more of children in one cluster at every k, in all five samples; the other clusters had 2–43 children sharing one rare answer. Its best silhouette (0.29 at k = 2, a 9,990 vs 10 split) is meaningless.

**Complete linkage, W0, part 0:**

| k | Mean silhouette | Smallest cluster (of 10,000) | Mean ARI (min–max) |
|---|---|---|---|
| 2 | 0.199 | 2,594 | 0.558 (0.263–0.760) |
| **3** | **0.208** | **573** | **0.525 (0.302–0.805)** |
| 4 | 0.122 | 573 | 0.350 (0.286–0.510) |

**k = 3 was approved on 2026-11-10:** highest silhouette, no cluster under 1%, acceptable stability, and unlike k = 2 (an amenity-only split) it separates an older group. The k = 2 vs 3 silhouette gap (0.009) is within the 0.02 practical guideline, so the choice rests on interpretability and stability.

## 5. What do the three clusters look like?

| | **Cluster 1:** amenities present | **Cluster 2:** few amenities | **Cluster 3:** older, higher reported treatment and screen time |
|---|---|---|---|
| Children (all 51,375) | 33,714 (65.6%) | 14,477 (28.2%) | 3,184 (6.2%) |
| In the 10,000 sample | 6,833 | 2,594 | 573 |
| Share weighted by `fwc` | 64.4% | 30.5% | 5.1% |
| Mean age | 9.6 | 8.9 | 13.2 |
| Ages 12–17 form | 42% | 37% | 72% |
| Park / recreation center / library | 97% / 69% / 96% | 37% / 6% / 12% | 46% / 10% / 22% |
| Mental-health treatment (parent-reported) | 13% | 2% | 78% |
| Medication (parent-reported) | 9% | 2% | 62% |
| Screen time 4+ hours | 22.5% | 21.4% | 51.3% |
| Screen time 4+ hours, weighted | 21.4% | 20.8% | 52.1% |
| Curiosity in learning "always" | 55% | 56% | 25% |
| Sleep below age minimum | 30% | 32% | 42% |
| Per-cluster silhouette (sample) | 0.231 | 0.179 | **0.064** |

Unweighted unless stated. Weighted values have no standard errors. All statistics: `outputs/cluster_profiles.csv`.

- Clusters 1 and 2 differ mainly in **neighborhood amenities**; their screen time is similar.
- Cluster 3 has much higher **reported treatment, medication and 4+ hour screen time**, but it is also the **oldest**, and age is a feature. Its screen time cannot be attributed to treatment or medication.
- Cluster 3 is the least well defined: 32% of its sample members have a negative silhouette.

## 6. How good and how robust is the model?

- **Weak structure.** Mean silhouette 0.208; no complete-linkage run exceeds about 0.27.
- **Assignment** of 41,375 children: no exact ties. Margin to the runner-up below 0.005 / 0.01 / 0.02 for 1.9% / 3.3% / 5.9% (flagged, not removed).
- **Leave-one-out:** 89.2% of sample children return to their cluster (cluster 1: 91.6%, 2: 87.5%, **3: 67.7%**). Assigned cluster-3 children look more extreme than sample members (64% vs 53% medication).

**Sensitivity runs** (ARI against the primary labeling, full data, k = 3; replicate band 0.30–0.81):

| Run | ARI | Inside band? |
|---|---|---|
| Domain-balanced weights (W1) | 0.082 | No |
| Neighborhood items down-weighted (W2: 1/3 each) | 0.042 | No |
| Neighborhood items as one 0–3 count | 0.044 | No |
| All binaries symmetric | 0.277 | No |
| With `eyedoctor` | 0.354 | Yes |
| Without the sleep feature | 0.481 | Yes |
| Complete cases only | 0.372 | Yes |

- **The cluster 1 vs 2 contrast depends on how the three correlated neighborhood items are weighted or coded.** A high-treatment, high-medication group reappears under W1, W2 and the count variant only, and its size changes between samples.
- **The variants are not more stable:** mean replicate ARI at k = 3 is 0.53 (primary), 0.36 (W2), 0.33 (count). The primary model is kept.
- **k = 4 with the count variant** was the most stable cell tested (mean ARI 0.54), but it splits mainly by sex and keeps no neighborhood information. Not adopted; details in [`docs/challenges.md`](../docs/challenges.md).

## 7. What are the limitations?

- Observational, cross-sectional, mostly parent-reported data: associations only.
- Unweighted clustering of an oversampled survey; weighted standard errors were not computed.
- The clusters are a descriptive partition of a continuum and change with weighting and coding.
- Mode imputation inflates common answers (complete-case run in the table above).
- `sleep_below_minimum` overlaps with age and is undefined for 446 infants under 4 months (excluded pairwise).
- Family-structure codes and screen-time levels 2–4 are shown by code number; labels were not checked against the codebook here.

## 8. How to use the outputs

| File | Use |
|---|---|
| `outputs/cluster_assignments.csv` | One row per child: `cluster` (1 = largest), `in_clustering_sample`, `assignment_method` (`hierarchical` / `assigned`), `assignment_margin`, ambiguity flags at 0.005 / 0.01 / 0.02, `replicate_part` |
| `outputs/cluster_profiles.csv` | Unweighted and weighted profiles with `n_used`; cells under 30 children are blank |
| `outputs/evaluation_results.csv`, `per_cluster_silhouette.csv`, `stability_sensitivity.csv` | Evidence for Sections 4–6 |
| `outputs/linkage_primary.npy`, `linkage_primary_hhid.csv` | The tree, for dendrograms |
| `outputs/model_config.json` | Specification, seed, package versions, SciPy-to-final label mapping |

When presenting clusters:
- Count both `hierarchical` and `assigned` children, or say which you used.
- Say "associated with" and "parent-reported medication use", never "causes" or "diagnosed".
- Show age next to any screen-time comparison for cluster 3, and state the weak silhouette and weighting sensitivity.

## 9. References

- U.S. Census Bureau. *2024 National Survey of Children's Health, Topical Public Use File* [Data set]. Accessed 21 September 2026. https://www.census.gov/programs-surveys/nsch/data/datasets/nsch2024.html
- U.S. Census Bureau. [*2024 NSCH Topical Variable List*](https://www2.census.gov/programs-surveys/nsch/technical-documentation/codebook/2024-NSCH-Topical-Variable-List.pdf) (codebook) and [*NSCH Analytic Guide*](https://www2.census.gov/programs-surveys/nsch/technical-documentation/methodology/NSCH-Analytic-Guide.pdf).
- Gower, J. C. (1971). A general coefficient of similarity and some of its properties. *Biometrics*, 27(4), 857–871.
- Kaufman, L., & Rousseeuw, P. J. (1990). *Finding Groups in Data: An Introduction to Cluster Analysis.* Wiley.
- Rousseeuw, P. J. (1987). Silhouettes: a graphical aid to the interpretation and validation of cluster analysis. *Journal of Computational and Applied Mathematics*, 20, 53–65.
- Hubert, L., & Arabie, P. (1985). Comparing partitions. *Journal of Classification*, 2(1), 193–218.
- Paruthi, S., et al. (2016). Recommended amount of sleep for pediatric populations. *Journal of Clinical Sleep Medicine*, 12(6), 785–786.
- Virtanen, P., et al. (2020). SciPy 1.0. *Nature Methods*, 17, 261–272.
- Pedregosa, F., et al. (2011). Scikit-learn: machine learning in Python. *Journal of Machine Learning Research*, 12, 2825–2830.
