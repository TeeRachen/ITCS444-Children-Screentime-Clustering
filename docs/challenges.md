# Challenges, Failed Attempts and Lessons Learned

A running log of what did not work, what had to be changed, and what is still unresolved. It is kept so that the final report can state the limits of the analysis honestly. Entries describe the project's own work; results are associations only.

The main README gives a short summary. The model and its results are in [`../reports/model_summary.md`](../reports/model_summary.md).

## 1. Variables that could not be used as planned

| Variable | Problem | Outcome |
|---|---|---|
| `physactiv` | Not asked for ages 0–5. | Dropped. |
| Height and weight | Not asked for ages 0–5. | Excluded before candidate selection. |
| `focuson` | Asked only for ages 3–5. | Excluded before candidate selection. |
| `eyecare2` | Asked only if the child saw an eye doctor. | Excluded before candidate selection. |
| `sleep_hours` (the two sleep items combined) | The T1 item is 24-hour sleep including naps, and the T2/T3 item is weeknight sleep, with different hour anchors. The combined codes are not comparable across forms and show an artificial jump at the age 5→6 boundary. | Kept for descriptive EDA only. Replaced in the model by the derived `sleep_below_minimum`. |
| `eyedoctor` | T1 asks "ever", T2/T3 ask "in the past 2 years". These cannot be reconciled for ages 2–5. | Excluded from the primary model; kept for a sensitivity run. |

## 2. Data and implementation difficulties

- **Sleep indicator is a compromise.** `sleep_below_minimum` reduces sleep to two levels, changes sharply where the guideline changes with age, overlaps partly with age (also a feature), and is undefined for 446 infants under 4 months (structural missingness, not nonresponse).
- **Mode imputation** fills 0.4–2.7% nonresponse per variable with the most common answer, which inflates the most common category. 4,115 children have at least one imputed value; a complete-case sensitivity run was done (see the sensitivity table below).
- **No ready-made Gower package fit.** The PyPI `gower` package has no ordinal or asymmetric-binary type and would treat integer-coded nominal variables as numeric, so Gower distance was implemented directly in NumPy and validated against hand-worked examples and a pair-by-pair reference.
- **Full-data clustering does not fit in memory.** A pairwise distance matrix for 51,375 children is about 9.8 GiB, so clustering runs on a stratified sample of 10,000 children, and the remaining children are assigned afterwards. Those children did not shape the clusters.
- **SciPy's `cophenet(Z, Y)` needs about 2.3 GiB at n = 10,000.** It was replaced with a chunked implementation, checked to match SciPy exactly at n = 2,000.
- **Limited free RAM on the project machine.** The machine has 13.8 GiB, but often only 0.7–1.4 GiB was free because of other applications. A memory check (`require_memory`) stops each large step before it allocates, and the 10,000-child runs were done only after applications were closed (peak process memory 1.5 GiB).
- **Environment mismatch.** The system Python on PATH has no pandas, and the Jupyter kernel pointed to it, so `jupyter nbconvert --execute` failed. Notebooks are run with the Anaconda interpreter instead (see "Run it" in the README).
- **Notebook housekeeping.** `01_preprocessing.ipynb` declares nbformat 4.0 but contains cell ids, so `nbformat.validate` fails (harmless; not yet fixed). The saved outputs in `02_eda.ipynb` were produced before the sleep imputation fix and need a re-run.
- **Repository renamed** after the README was first written (now `ITCS444-Children-Screentime-Clustering`), so earlier links needed updating.

## 3. Clustering attempts that did not work as hoped

*(Results below come from `03_clustering.ipynb`, executed on 2026-11-10 and 2026-11-11. Complete linkage, W0 and k = 3 were approved as the primary model on 2026-11-10. Items 5 to 9 explain why it is kept despite its weaknesses.)*

1. **Average linkage, the planned primary method, is degenerate.**
   - Seen first in the 2,000-child pilot and again at 10,000 in all five replicate samples.
   - One cluster holds 99% or more of the children at every k from 2 to 10; the other clusters have 2–43 children each and are defined by a rare answer (for example, mental-health care needed but not received, or vision difficulty).
   - Its best silhouette (0.29 at k = 2, a 9,990 vs 10 split) is an artefact, not evidence of structure.
2. **Complete linkage is usable but the structure is weak.**
   - Mean silhouette is at most 0.21 for any k (0.199 at k = 2, 0.208 at k = 3, 0.09–0.12 for k ≥ 5), which is below the range usually read as meaningful structure. Cophenetic correlation is 0.48, compared with 0.64 for average linkage.
   - Stability across the five replicate samples is moderate: mean pairwise ARI 0.56 at k = 2 and 0.53 at k = 3, falling to about 0.32–0.40 for larger k.
3. **Results are sensitive to weighting.**
   - Under domain-balanced weights (W1), complete linkage differs from equal weights (W0) by more than the stability band at k = 2–5.
   - The three correlated neighborhood items drive the split under W0, and sex drives it under W1. A W2 run was therefore added.
   - A group with high rates of mental-health treatment and medication and more screen time appears under W0, W1 and W2 (and with the neighborhood count), but not in the other four sensitivity runs (see item 5).
4. **Silhouettes from different weightings cannot be compared directly.** W0 and W1 results are compared by ARI, cluster sizes, profiles and cross-evaluation instead.
5. **The approved primary model is not robust.** Sensitivity runs with W1, W2, or one count in place of the three neighborhood items agree with the primary clusters at ARI 0.04-0.08, far below the replicate-sample band (0.30-0.81 at k = 3). A group with high reported mental-health treatment and medication use reappears in those three runs but not in the other four (all binaries symmetric, with `eyedoctor`, without sleep, complete cases).
6. **The assignment rule reproduces the smallest cluster only partly.** In a leave-one-out check, 67.7% of the 573 sample children in cluster 3 return to it (89.2% overall), and assigned cluster-3 children look more extreme than sample members.
7. **The neighborhood-item variants did not fix the instability.** Despite higher silhouettes on part 0, the W2 and count variants were not more stable across replicate samples at k = 3 (mean ARI 0.36 and 0.33 vs 0.53 for the primary model), and the high-medication group changed size and medication share from part to part.
8. **Alternative k = 4 test (exploratory, not adopted).** The neighborhood-count variant at k = 4 was the most stable cell tested (mean ARI 0.54, smallest cluster at least 583 children in every part), so it was profiled on part 0 and all children. It was run in a scratch script outside the notebook, so its numbers are not in `outputs/`. Two clusters split children by sex (1% vs 100% female; 4+ hour screen time 22.4% vs 18.6%), and two split the older high-treatment group by sleep below the minimum (2% vs 100%). Mean neighborhood amenities were about 1.9 in all four clusters, so the environment domain no longer separated anything, and agreement with the primary k = 3 labelling was ARI 0.05. Its stability comes mostly from the easy-to-reproduce sex split, so it was not adopted.
9. **Feature-engineering experiment (2026-11-11, not adopted).** Four candidates were run under a plan fixed beforehand (`docs/feature_engineering_plan.md`, `04_feature_engineering.ipynb`, `reports/feature_engineering.md`): A without sex, B with `k4q22_r` and `k4q23` merged, C combining A, B and the neighborhood count, D = C without sleep.
   - At k = 3 none met the four decision rules. Mean pairwise ARI across replicate samples was 0.23 (A), 0.35 (B), 0.28 (C) and 0.49 (D), against 0.525 for the primary and the 0.575 needed. None of the four had a W0-vs-W1 agreement inside its own replicate band.
   - C and D had clusters under 1% of the sample in some parts (37 and 51 children). A, B and C each had a feature almost defining a cluster split (Cramér's V 0.86-0.93); the primary's own value is 0.84 for the library item.
   - Removing features moved the split to other binary items (sex, sleep, amenities) instead of fixing the weak structure. The primary model is kept.

## 4. Open limitations and unresolved decisions

- **Weak cluster separation.** The children look more like a continuum than distinct groups. Clusters are a descriptive partition and should not be presented as natural categories.
- **Binary features dominate Gower distances.** Correlated yes/no items (the three neighborhood amenities) dominate the primary split. A single 0-3 count and W2 were tried; neither was more stable than the primary model at k = 3, so the primary model is kept. The count variant at k = 4 was more stable (mean ARI 0.54) and could be explored if the project wants a different k; that needs a new decision.
- **Cluster 3 is older.** Its higher screen time cannot be separated from age with these clusters.
- **Unweighted clustering of an oversampled survey.** Clusters describe the survey sample, not all U.S. children. Weighted standard errors would need a survey package that has not been approved.
- **Parent-reported, cross-sectional data.** No causal claims can be made. `k4q23` is parent-reported medication use, not a diagnosis.
- **W2 uses the project's own definition** (the three neighborhood items together carry one feature's weight), not one taken from a reference, and the repository is unlicensed for now.
- **Family-structure and screen-time codes** are reported by code number in the profiles; `family_r` labels were not checked against the codebook.

---

## Sensitivity runs and replicate stability (complete linkage, full-data labellings)

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

**Consequence:** the approved primary model is a descriptive partition that is sensitive to modelling choices, not a stable set of groups. The variants do not offer a more stable alternative at k = 3, so the primary model is kept, with this limitation stated. An exploratory k = 4 test of the count variant is described in item 8 above.
