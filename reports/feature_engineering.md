# Feature-engineering experiment: findings

**Status:** experiment finished 2026-11-11. The approved primary model (complete linkage, W0, k = 3, 14 features), `outputs/`, the README and `reports/model_summary.md` are **unchanged**. Only the user decides whether to change the primary model.
**Plan:** `docs/feature_engineering_plan.md` (fixed before running). **Notebook:** `04_feature_engineering.ipynb`. **Results:** `outputs/feature_engineering/` (CSV files only, no distance matrices).
All results are associations in the 2024 NSCH sample, clustered unweighted; they do not describe all U.S. children. Medication use is parent-reported, not a diagnosis.

## Result in one paragraph

**None of the four candidates meets decision rules 1-4, so none is "worth adopting".** No candidate reaches the stability bar (mean pairwise ARI >= 0.575; the best, D, is 0.493 and the primary itself is 0.525). None is less weighting-sensitive than its own replicate band allows. Candidates C and D produce clusters under 1% of the sample in some parts, and A, B and C each have a feature that almost defines a cluster split (Cramér's V >= 0.8). The primary also fails rule 4 (see below), so the cleaning did not fix the problem; it moved it. The recommendation is to **keep the approved primary model** and report this experiment as a tested negative result.

## Step 1: diagnostics (no clustering)

- **Association between features:** the largest pairs are `k4q22_r` x `k4q23` (Cramér's V 0.50), the three neighborhood items with each other (V 0.40-0.48), age x screen time (Spearman 0.50, a developmental pattern, not redundancy). Everything else is V < 0.3.
- **Share of the total pairwise Gower distance (part 0, equal weights):** `family_r` 11.2%, `sc_sex` 11.1%, `k10q13` 11.1%, `k10q14` 9.7%, `sleep_below_minimum` 9.4%, `screentime` 8.4%, `k10q12` 7.9%, age 7.8%, `k4q22_r` 6.3%, `k6q71_r` 5.6%, `k4q23` 3.9%, `foodsit` 3.6%, `higrade` 3.3%, `blindness` 0.6%. By domain: environment 35.6% (the three neighborhood items alone 28.7%), general 30.2%, lifestyle 23.4%, health 10.8%. The asymmetric binaries carry little distance because most pairs share "No".
- **Mental-health items** reproduce the planned counts (none 41,253; unmet need only 978; treatment only 3,789; medication without treatment 1,924; both 3,431).
- **No extra candidate** was added (reasons are in the notebook, written before clustering).

## Step 3: decision-rule table (k = 3, complete linkage, W0)

| | Primary (ref.) | A no sex (13) | B merged mental health (13) | C combined (10) | D C without sleep (9) |
|---|---|---|---|---|---|
| R1 mean pairwise ARI (need >= 0.575) | 0.525 | 0.233 | 0.354 | 0.281 | 0.493 |
| (min / max) | 0.302 / 0.805 | 0.071 / 0.472 | 0.209 / 0.539 | 0.139 / 0.677 | 0.190 / 0.890 |
| R2 ARI W0 vs W1 (need >= own band min) | 0.082 (min 0.302) | 0.021 (0.071) | 0.009 (0.209) | 0.121 (0.139) | 0.085 (0.190) |
| R3 smallest cluster in any part (need >= 100 of 10,000) | 271 | 193 | 907 | **37** | **51** |
| R4 largest Cramér's V with a clustering feature, part 0 (need < 0.8) | 0.84 (`k10q14`) | 0.86 (`k10q13`) | 0.86 (`k10q13`) | 0.93 (`sleep_below_minimum`) | 0.52 (`mh_care`) |
| R1 / R2 / R3 / R4 met? | no / no / yes / no | no / no / yes / no | no / no / yes / no | no / no / **no** / no | no / no / **no** / yes |
| Verdict | reference | tested, not adopted | tested, not adopted | tested, not adopted | tested, not adopted |

- **R2:** every candidate is below its own band minimum, but C (0.121 vs 0.139) is close. C's band is also very low (min 0.139), so its "closeness" reflects weak stability, not weighting robustness.
- **R3:** C has a cluster of 37 children in part 1; D has clusters of 61 (part 1) and 51 (part 4). Counts in the other parts are above 100.
- **R4 (judged on part 0, as fixed before the run):** the highest part-0 V is reported above. Across all five parts the highest is 0.84 (primary), 0.87 (A), 0.86 (B), 0.93 (C), 0.95 (D, `mh_care` in part 2). So D passes rule 4 on part 0 but would not if all parts were counted. D fails rules 1-3 regardless.
- **The primary row** was re-run through this notebook's pipeline: its k = 3 labelling equals `outputs/cluster_assignments.csv` exactly (ARI 1.0) and its stability and W1 numbers match notebook 03. The primary's own rule 4 value (V 0.84 with `k10q14`, i.e. the library item) shows the current split is largely driven by one neighborhood item; this was not a decision rule when the primary was approved.

## What the candidates did (profiles, k = 3, part 0, all children; associations only)

- **A (no sex):** the 3 clusters are now split by neighborhood amenities (recreation center present in 79-95% of two clusters, 0.4% in the third) and by sleep and age (cluster 2: mean age 12.1, 6% below the sleep minimum; cluster 3: mean age 7.9, 56% below). The high mental-health-treatment group (cluster 2: 30% treatment, 21% medication) is blended with older age. Sex is no longer a driver but amenity items now dominate (V 0.86).
- **B (merged):** one cluster is 95% female and one is 7% female (sex becomes the split, V 0.60 with `sc_sex`), with amenities (V 0.86). The merged mental-health item did not produce a mental-health cluster at k = 3 (treatment 12-18% and medication 9-13% in all three clusters).
- **C (combined):** the split follows sleep below the minimum (99.8% in one cluster, 2-5% in the others; V 0.93 in part 0) and age (mean ages 7.0 / 10.5 / 13.3). Cluster membership by age and screen time is plausible, but the sleep feature takes over the split.
- **D (no sleep):** clusters follow age (mean 7.0 / 13.4 / 12.9), screen time and mental-health care; cluster 3 (2,286 children) has 64% treatment, 67% medication and 53% screen time 4+ h. It is the most stable candidate (R1 0.493; k = 2 mean ARI 0.579) but has tiny clusters in some parts and its k = 3 solution changes by the replicate band (ARI min 0.19).

## Other observations (informational, not decision rules)

- Silhouettes are each candidate's own and are **not compared across candidates**: at k = 3, part 0, W0: primary 0.208, A 0.111, B 0.185, C 0.126, D 0.121. All are weak (below 0.25).
- ARI of each candidate's part-0 labelling against the primary at k = 3: A 0.158, B 0.446, C 0.018, D 0.043. B keeps part of the primary structure (amenities); C and D are essentially unrelated to it.
- Under domain-balanced weights (W1) the candidates' silhouettes rise (e.g. C 0.313, B 0.289) but the labelling changes completely (ARI vs W0 of 0.01-0.12), the same sensitivity the primary shows.
- Other k (2, 4, 5, 6) are informational only; k was not chosen per candidate. Mean pairwise ARI at k = 2 is 0.558 for the primary and 0.100 / 0.322 / 0.226 / 0.579 for A-D.
- Assignment margins below 0.01 for k = 3: primary 7.7%, A 11.3%, B 7.1%, C 9.7%, D 5.1% of the assigned children (the share averaged over the five parts).

## Method notes and caveats

- Same seed (42), same five disjoint parts (verified identical to notebook 03), complete linkage, W0 Gower, assignment of the other 41,375 children to the nearest cluster by mean distance. W1 runs on part 0 only.
- **Tied merge heights:** in one informational run (candidate D, part 0, k = 5) the height cut returned only 4 clusters because of tied heights; the notebook then undid the last k - 1 merges in linkage order (flag `tie_fallback_cut` in `candidate_evaluation.csv`). It affects no decision-rule number (k = 3 cuts were height cuts in all runs).
- Cramér's V is the uncorrected statistic with every feature treated as categorical (age as 18 levels). These choices and the part-0 reading of rule 4 were fixed before the run.
- Interpretations of clusters are descriptions of the sample, not causal claims.

## Recommendation

Keep the approved primary (complete linkage, W0, k = 3). The experiment supports the existing limitations: the clusters are weak, mainly reflect which binary items (neighborhood amenities, sleep, sex) are included, and change with weighting. One pattern recurs: an older group with more screen time and higher parent-reported mental-health treatment and medication. It is sharply defined in the primary (78% treatment, 62% medication, 3,184 children) and in D (cluster 3: 64% treatment, 67% medication, 53% screen time 4+ h, 2,286 children), diluted in A and C (30-32% treatment, 21-24% medication) and absent in B at k = 3, where the splits are by sex and amenities. Whether to cite this experiment in the report, or to try a methodological change (for example, a different clustering approach for the binary items), is the user's decision.
