# Challenges, Failed Attempts and Lessons Learned

What did not work, what had to change, and what is still unresolved, so the limits of the analysis can be stated honestly. The model and its results are in [`../reports/model_summary.md`](../reports/model_summary.md); dated decisions are in [`../CHANGELOG.md`](../CHANGELOG.md).

## Contents

1. [Which variables could not be used as planned?](#1-which-variables-could-not-be-used-as-planned)
2. [What made the data and implementation hard?](#2-what-made-the-data-and-implementation-hard)
3. [Which clustering attempts did not work?](#3-which-clustering-attempts-did-not-work)
4. [What is still unresolved?](#4-what-is-still-unresolved)
5. [Sensitivity and variant stability in detail](#5-sensitivity-and-variant-stability-in-detail)

## 1. Which variables could not be used as planned?

| Variable | Problem | Outcome |
|---|---|---|
| `physactiv` | Not asked for ages 0–5 | Dropped |
| Height and weight | Not asked for ages 0–5 | Excluded before candidate selection |
| `focuson` | Asked only for ages 3–5 | Excluded before candidate selection |
| `eyecare2` | Asked only if the child saw an eye doctor | Excluded before candidate selection |
| `sleep_hours` | T1 = 24-hour sleep incl. naps, T2/T3 = weeknight sleep, different hour anchors; artificial jump at age 5→6 | EDA only; the model uses `sleep_below_minimum` |
| `eyedoctor` | T1 asks "ever", T2/T3 "in the past 2 years"; irreconcilable for ages 2–5 | Excluded from the primary model; sensitivity run only |

## 2. What made the data and implementation hard?

- **Sleep indicator is a compromise.** Two levels only, jumps where the guideline changes with age, overlaps with age, undefined for 446 infants under 4 months (structural, not nonresponse).
- **Mode imputation** inflates the most common answer. 4,115 children have at least one imputed value; a complete-case sensitivity run was done.
- **No Gower package fit.** PyPI `gower` has no ordinal or asymmetric type and treats integer nominal codes as numeric, so Gower was written in NumPy and validated against hand-worked examples and a pair-by-pair reference.
- **Full-data clustering does not fit in memory** (about 9.8 GiB), so 10,000 children are clustered and the rest assigned; assigned children did not shape the clusters.
- **SciPy `cophenet(Z, Y)` needs about 2.3 GiB at n = 10,000.** Replaced by a chunked version, checked to match SciPy exactly at n = 2,000.
- **Little free RAM** (often 0.7–1.4 of 13.8 GiB). `require_memory` stops each large step before allocating; full runs were done with other apps closed (peak 1.5 GiB).
- **Environment mismatch.** The system Python has no pandas and the Jupyter kernel pointed to it, so notebooks are run with the Anaconda interpreter.
- **Housekeeping.** `01_preprocessing.ipynb` declares nbformat 4.0 but has cell ids, so `nbformat.validate` fails (harmless, not fixed). `02_eda.ipynb` outputs predate the sleep-imputation fix and need a re-run.

## 3. Which clustering attempts did not work?

*Source: `03_clustering.ipynb` (2026-11-10/11) and `04_feature_engineering.ipynb`. Items 5–9 explain why the primary model is kept despite its weaknesses.*

1. **Average linkage, the planned primary method, is degenerate.** In the pilot and in all five 10,000-child samples, one cluster holds 99% or more of children at every k = 2–10; the others have 2–43 children sharing one rare answer (e.g. care needed but not received, vision difficulty). Its 0.29 silhouette at k = 2 (9,990 vs 10) is an artefact.
2. **Complete linkage works, but the structure is weak.** Silhouette at most 0.21 (0.199 at k = 2, 0.208 at k = 3, 0.09–0.12 for k ≥ 5); cophenetic correlation 0.48 vs 0.64 for average. Mean replicate ARI 0.56 (k = 2), 0.53 (k = 3), about 0.32–0.40 for larger k.
3. **Results depend on weighting.** W1 differs from W0 by more than the stability band at k = 2–5: the three neighborhood items drive the W0 split, sex drives W1. A W2 run was added.
4. **Silhouettes from different weightings are not comparable**, so W0 and W1 are compared by ARI, sizes, profiles and cross-evaluation.
5. **The primary model is not robust.** W1, W2 and the neighborhood count agree with it at ARI 0.04–0.08, far below the band (0.30–0.81). A high-treatment, high-medication group reappears in those three runs only (Section 5).
6. **Assignment reproduces the smallest cluster only partly.** Leave-one-out: 67.7% of cluster 3's 573 sample children return to it (89.2% overall); assigned cluster-3 children look more extreme.
7. **Neighborhood variants did not fix the instability.** Despite higher part-0 silhouettes, W2 and the count are less stable at k = 3 (mean ARI 0.36 and 0.33 vs 0.53), and the high-medication group changes size between parts.
8. **k = 4 with the count variant (exploratory, not adopted).** The most stable cell tested (mean ARI 0.54, smallest cluster ≥ 583), run in a scratch script, so not in `outputs/`. Two clusters split by sex (1% vs 100% female; 4+ h 22.4% vs 18.6%), two split the older high-treatment group by sleep (2% vs 100% below minimum). About 1.9 amenities in every cluster, ARI 0.05 vs the primary. Its stability comes from the easy sex split.
9. **Feature-engineering experiment (not adopted).** Four candidates under rules fixed in advance (A no sex, B merged mental-health items, C = A + B + neighborhood count, D = C without sleep). Mean ARI 0.23 / 0.35 / 0.28 / 0.49 vs 0.525 and the 0.575 needed; none passed the W0-vs-W1 rule; C and D had clusters of 37 and 51 children; A–C had a near-defining feature (V 0.86–0.93; primary 0.84 for the library item). Removing features moved the split to other binary items. See [`../reports/feature_engineering.md`](../reports/feature_engineering.md).

## 4. What is still unresolved?

- **Weak separation.** Children look more like a continuum than distinct groups; clusters are a descriptive partition.
- **Binary features dominate Gower distances**, especially the three correlated amenity items. The count variant at k = 4 is more stable and could be explored if the project wants a different k (needs a new decision).
- **Cluster 3 is older**, so its higher screen time cannot be separated from age.
- **Unweighted clustering of an oversampled survey**; weighted standard errors need a survey package that has not been approved.
- **Parent-reported, cross-sectional data**: no causal claims; `k4q23` is medication use, not a diagnosis.
- **W2 is the project's own definition** (the three neighborhood items together carry one feature's weight).
- **Family-structure and screen-time codes** are shown by number; `family_r` labels were not checked against the codebook.

## 5. Sensitivity and variant stability in detail

**ARI of each run against the primary labeling** (complete linkage, full data, k = 3; band 0.302–0.805):

| Run | ARI | Inside band? | Highest-medication group: share of children, % medication |
|---|---|---|---|
| W1 (domain-balanced) | 0.082 | No | 10.7%, 65% |
| W2 (neighborhood items 1/3 each) | 0.042 | No | 12.0%, 61% |
| Neighborhood count (0–3) | 0.044 | No | 12.2%, 58% |
| All binaries symmetric | 0.277 | No | 36.6%, 17% |
| With `eyedoctor` | 0.354 | Yes | 24.3%, 19% |
| Without sleep | 0.481 | Yes | 14.5%, 20% |
| Complete cases | 0.372 | Yes | 29.1%, 11% |

- The high-medication group (about 11–12% of children, 58–65% medication, 41–47% with 4+ h) appears only with W1, W2 and the count.
- W2 and the count have higher part-0 silhouettes (0.223, 0.218) and cophenetic correlations (0.532, 0.555), but these are not comparable across distance definitions, so their stability was checked:

**Replicate stability of the variants** (10 pairwise ARIs per cell; mean, minimum in brackets):

| k | Primary (W0) | W2 | Neighborhood count |
|---|---|---|---|
| 2 | 0.558 (0.263) | 0.117 (−0.022) | 0.246 (−0.002) |
| 3 | 0.525 (0.302) | 0.357 (0.087) | 0.325 (0.099) |
| 4 | 0.350 (0.286) | 0.346 (0.121) | 0.539 (0.213) |

- At k = 3 neither variant beats the primary; at k = 2 both are clearly worse. Only the count at k = 4 is more stable (item 8).
- The high-medication group varies across parts: 8–22% of children with 32–62% medication (count), 9–13% with 44–67% (W2); its 4+ h share stays at 43–50%.
