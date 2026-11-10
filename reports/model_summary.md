# Model Summary: Children's Screen-Time Clustering

**Course:** ITCS444 Data Mining
**Data:** 2024 National Survey of Children's Health (NSCH), Topical Public Use File, U.S. Census Bureau
**Source of every number below:** `03_clustering.ipynb` (executed 2026-11-10 and 2026-11-11) and the files in `outputs/`
**Status:** model fitted and evaluated; interpretation not yet reviewed for the final report

> **How to read this document.** The clusters describe **associations in a survey sample**. They are not causes, not diagnoses and not natural categories of children. The structure is weak (silhouette about 0.2) and changes with modelling choices (Section 5). Use the clusters to describe tendencies of groups, never to describe or label an individual child.

---

## 1. Question and approach

**Research question:** what health, lifestyle and environmental patterns are associated with children's screen-time levels?

Children with similar characteristics are grouped by hierarchical clustering. Screen time is one of the clustering **features**, not an outcome that is predicted. The groups are then described, including how screen time varies across them.

## 2. Data and features

51,375 children aged 0-17 (one per household). Questionnaire form T1 = ages 0-5, T2 = 6-11, T3 = 12-17. The survey oversamples ages 0-5 and children with special health care needs, and the clustering is **unweighted**, so it describes the survey sample, not all U.S. children.

**14 features** (preprocessing: mode imputation for 0.4-2.7% nonresponse per variable; 47,391 children have no imputed value in these features):

| Domain | Features |
|---|---|
| General | age, sex, family structure |
| Health | vision difficulty even with glasses, mental-health treatment in the past 12 months (parent-reported), medication for emotions, concentration or behavior in the past 12 months (parent-reported; **not a diagnosis**) |
| Lifestyle | screen time (5 levels, weekday), sleep below the age-specific recommended minimum, curiosity in learning |
| Environment | highest adult education, food affordability, park or playground / recreation center / library in the neighborhood |

Not used: `eyedoctor` (worded differently by form), the original sleep-hours codes (not comparable across forms), height, weight, `focuson`, `eyecare2`, `physactiv` (not asked for all ages).

## 3. Method

| Step | Choice |
|---|---|
| Distance | **Gower distance**, implemented in NumPy and validated against hand-worked examples and a pair-by-pair reference (exact match). Ordinal and numeric features: range-normalized difference; nominal and binary: simple matching; asymmetric binaries (vision difficulty, medication): two "No" answers are not counted as similar. |
| Weights | **Equal weights (W0).** |
| Clustering | **Hierarchical, complete linkage**, k = 3. |
| Sample | A stratified sample of **10,000 children** (strata: questionnaire form x screen-time level; seed 42) is clustered. The other **41,375 children are assigned** to the cluster with the smallest average distance to its sample members. |
| Evaluation | Mean and per-cluster silhouette (k = 2-10), cluster sizes, stability (Adjusted Rand Index across five disjoint stratified replicate samples), sensitivity runs. |

A full 51,375 x 51,375 distance matrix (about 9.8 GiB) does not fit in memory on the project machine, which is why the sample-and-assign design is used.

## 4. Why complete linkage and k = 3

**Average linkage was rejected.** At 10,000 children it put 99% or more of the children in one cluster at every k, in all five replicate samples. The other clusters had 2-43 children defined by a single rare answer. Its best silhouette (0.29 at k = 2, a 9,990 vs 10 split) is not meaningful.

**Complete linkage, W0, part 0:**

| k | Mean silhouette | Smallest cluster (of 10,000) | Stability: mean ARI (min-max) |
|---|---|---|---|
| 2 | 0.199 | 2,594 | 0.558 (0.263-0.760) |
| **3** | **0.208** | **573** | **0.525 (0.302-0.805)** |
| 4 | 0.122 | 573 | 0.350 (0.286-0.510) |

**k = 3 was approved** (2026-11-10) because its silhouette is the highest, it has no cluster under 1%, its stability is acceptable, and, unlike k = 2, it separates the older group described below. k = 2 is more stable but splits children only by neighborhood amenities. The difference between k = 2 and k = 3 (0.009) is within the 0.02 practical guideline, so the choice rests on interpretability and stability, not on the silhouette.

## 5. Results (k = 3)

Neutral working names, based only on how the clusters differ:

| | **Cluster 1:** neighborhood amenities present | **Cluster 2:** few neighborhood amenities | **Cluster 3:** older, higher reported treatment and screen time |
|---|---|---|---|
| Children (all 51,375) | 33,714 (65.6%) | 14,477 (28.2%) | 3,184 (6.2%) |
| In the 10,000 clustering sample | 6,833 | 2,594 | 573 |
| Share weighted by `fwc` | 64.4% | 30.5% | 5.1% |
| Mean age | 9.6 | 8.9 | 13.2 |
| Ages 12-17 questionnaire | 42% | 37% | 72% |
| Park / recreation center / library | 97% / 69% / 96% | 37% / 6% / 12% | 46% / 10% / 22% |
| Mental-health treatment (parent-reported) | 13% | 2% | 78% |
| Medication for emotions, concentration or behavior (parent-reported) | 9% | 2% | 62% |
| Screen time 4+ hours | 22.5% | 21.4% | 51.3% |
| Screen time 4+ hours, weighted | 21.4% | 20.8% | 52.1% |
| Curiosity in learning "always" | 55% | 56% | 25% |
| Sleep below age minimum | 30% | 32% | 42% |
| Per-cluster silhouette (sample) | 0.231 | 0.179 | **0.064** |

Unweighted unless stated. Weighted values have no standard errors, so small differences should not be over-read. All profile statistics are in `outputs/cluster_profiles.csv`.

**Main points**
- Clusters 1 and 2 differ mainly in **neighborhood amenities**. Their screen-time distributions are similar.
- Cluster 3 has **much higher reported mental-health treatment, medication use and 4+ hour screen time**. It is also the **oldest** cluster, and screen time rises with age, so its higher screen time cannot be attributed to treatment or medication. Age is a clustering feature.
- Cluster 3 is the least well defined (silhouette 0.064; 32% of its sample members have a negative silhouette).

## 6. Quality and robustness

- **Weak structure.** Mean silhouette 0.208; no complete-linkage run exceeds about 0.27. Children form more of a continuum than separate groups.
- **Assignment** (41,375 children): no exact ties; margin to the second-closest cluster below 0.005 for 1.9%, below 0.01 for 3.3%, below 0.02 for 5.9% (flagged in `cluster_assignments.csv`, not removed). In a leave-one-out check, 89.2% of sample children return to their hierarchical cluster (cluster 1: 91.6%, cluster 2: 87.5%, **cluster 3: 67.7%**). Assigned children in cluster 3 look more extreme than sample members (64% vs 53% medication), so the rule describes that group more sharply than the tree does.
- **Sensitivity to modelling choices** (ARI of each run against the primary labelling, full data, k = 3; replicate-sample band 0.30-0.81):

| Run | ARI | Inside band? |
|---|---|---|
| Domain-balanced weights (W1) | 0.082 | No |
| Neighborhood items down-weighted (W2: 1/3 each) | 0.042 | No |
| Neighborhood items replaced by one 0-3 count | 0.044 | No |
| All binaries symmetric | 0.277 | No |
| With `eyedoctor` | 0.354 | Yes |
| Without the sleep feature | 0.481 | Yes |
| Complete cases only | 0.372 | Yes |

  The **neighborhood contrast between clusters 1 and 2 depends on how the three correlated neighborhood items are weighted or coded.** A group with high reported treatment and medication use reappears under W1, W2 and the count variant, but not in the other four runs, and its size and medication share change from sample to sample.
- **The two variants are not more stable.** Across the five replicate samples at k = 3, mean ARI is 0.53 for the primary model, 0.36 for W2 and 0.33 for the count variant, so the primary model was kept.
- **Alternative k = 4 test.** The count variant at k = 4 was the most stable cell tested (mean ARI 0.54), but it splits children mainly by sex and keeps no neighborhood information (about 1.9 amenities in every cluster). It was not adopted (README Section 15.3).

## 7. Limitations

- Observational, cross-sectional, mostly parent-reported data: associations only.
- Unweighted clustering of an oversampled survey; no generalization to all U.S. children.
- The clusters are a descriptive partition of a continuum; they change with weighting and coding choices.
- Mode imputation inflates common answers; complete-case results are in the sensitivity table.
- `sleep_below_minimum` is a two-level summary that overlaps with age and is undefined for 446 infants under 4 months (excluded pairwise, not imputed).
- Weighted standard errors were not computed (no survey package approved).
- Family-structure codes and screen-time levels 2-4 are shown by code number; labels were not checked against the codebook for this document.

## 8. Using the outputs (for visualization and insights)

| File | Use |
|---|---|
| `outputs/cluster_assignments.csv` | One row per child. `cluster` (1 = largest), `in_clustering_sample`, `assignment_method` (`hierarchical` / `assigned`), `assignment_margin` (blank for hierarchical), ambiguity flags at 0.005 / 0.01 / 0.02, `replicate_part`. |
| `outputs/cluster_profiles.csv` | Profiles, unweighted and weighted, with `n_used` per statistic. Statistics with fewer than 30 children are blank. |
| `outputs/evaluation_results.csv`, `per_cluster_silhouette.csv`, `stability_sensitivity.csv` | Evidence behind Sections 4-6. |
| `outputs/linkage_primary.npy`, `linkage_primary_hhid.csv` | The tree, for dendrograms. |
| `outputs/model_config.json` | Specification, seed, package versions, mapping of SciPy labels to final cluster numbers. |

**Suggestions for presenting the clusters**
- Always show **both** `hierarchical` and `assigned` children when counting, or say which you used.
- Say "associated with", never "causes"; say "parent-reported medication use", not "diagnosed".
- Show the age difference next to any screen-time comparison for cluster 3.
- State the weak silhouette and the sensitivity to weighting wherever the clusters are shown.
- Do not label children or families with cluster names; use the names above only as short descriptions of group tendencies.
