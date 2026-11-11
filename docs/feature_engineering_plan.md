# Feature-engineering experiment: fixed plan

**Branch:** `feature/feature-engineering` (created from the merged model-implementation work)
**Status:** plan fixed on 2026-11-11 **before running anything**. Do not change the candidates or decision rules after seeing results.
**Start here:** read `../PROJECT_CHECKPOINT.md` (one level above the repo folder) first, then this file. The checkpoint's section 2 has the environment notes and the `run_nb.py` helper; section 9 has the user's working rules.

## Goal
The approved primary model (complete linkage, W0, k = 3) gives weak clusters (silhouette 0.208) that change with weighting and with how the three correlated neighborhood items are coded (ARI vs primary 0.04-0.08 for W1, W2, neighborhood count). Test whether cleaning the feature set makes the clustering less sensitive and more stable. This is an experiment: **the primary model, `outputs/`, README and `reports/model_summary.md` are not changed**.

## Deliverables
- `04_feature_engineering.ipynb` (new). Copy the needed helpers from `03_clustering.ipynb` (Gower functions, monitoring, `stratified_disjoint_parts`, silhouette/ARI helpers, assignment helpers); do not edit `01`-`03` and do not import across notebooks. Use the same seed (42), the same five disjoint parts, complete linkage, k = 2-6.
- `outputs/feature_engineering/*.csv` (diagnostics and candidate results; never distance matrices).
- `reports/feature_engineering.md` (findings, decision-rule table, recommendation).
- `outputs/feature_engineering/progress.md` (a log updated after every finished step, so work can resume after an interruption).
- Update `PROJECT_CHECKPOINT.md` at the end.

## Step 1: diagnostics only (no clustering)
- Association matrix between the 14 features (Cramer's V for categorical, Spearman for ordinal).
- Each feature's share of the total pairwise Gower distance (mean per-feature contribution on part 0).
- Prevalence of each binary feature.
- Check the mental-health items: `k4q22_r` (1 received, 2 needed but not received, 3 not needed) x `k4q23` (1 medication, 2 none). Observed counts: none 41,253; unmet need only 978; treatment only 3,789; medication without treatment 1,924 (297 + 1,627); both 3,431.
- Diagnostics may add **at most one** extra candidate, written into the notebook **before** any clustering runs.

## Step 2: candidates (equal weights W0, complete linkage)
| | Candidate | Change from primary | Features |
|---|---|---|---|
| A | No sex | `sc_sex` moves to profiling only | 13 |
| B | Merged mental health | `k4q22_r` + `k4q23` become one nominal feature `mh_care` with 5 categories: 1 none, 2 unmet need only, 3 treatment only, 4 medication without treatment received, 5 both | 13 |
| C | Combined cleanup | A + B + `k10q12`/`k10q13`/`k10q14` replaced by one ordinal `amenity_count` (0-3; 1 = present) | 10 |
| D | C without sleep | C minus `sleep_below_minimum` (overlap with age) | 9 |

## Step 3: run and judge
For each candidate: all five parts (hierarchical on 10,000, assign the other 41,375 children), k = 2-6; pairwise ARI across the five full-data labellings (10 pairs); ARI against the primary labelling; per-cluster sizes; plus the same candidate with **domain-balanced weights (W1)** on part 0 only, compared with its own W0 labelling on the full data.

**Decision rules, judged at k = 3 only:**
1. More stable than the primary: mean pairwise ARI >= 0.575 (primary mean 0.525, min 0.302).
2. Less weighting-sensitive: ARI between its W0 and W1 full-data labellings is inside its own replicate band (>= the band minimum). The primary's W0-vs-W1 ARI was 0.082.
3. No tiny clusters: no cluster under 1% of the 10,000 sample in any part.
4. No single-feature split: no clustering feature has Cramer's V >= 0.8 with the cluster labels.
5. Silhouettes are reported but **not compared** across candidates (different distance definitions).
6. Other k values are informational; do not choose k per candidate.

A candidate meeting rules 1-4 is "worth adopting"; otherwise report it as tested. **Only the user decides** whether to change the primary model.

## Boundaries
- Do not commit or push (the user commits). Do not install packages. Do not edit `01`-`03`, the existing `outputs/*`, the README or `reports/model_summary.md`.
- Use the Anaconda interpreter (checkpoint section 2). Check free RAM before each run (each run peaks near 1.5 GiB). Stop and write up if RAM is too low.
- If a step fails twice, stop and report instead of improvising methodology.
- Profile clusters with associations in neutral, non-stigmatizing language; medication use is parent-reported, not a diagnosis.

## Time estimate
About 3 hours elapsed; compute about 45 minutes (about 8 minutes per candidate for five parts, plus W1 checks). Smoke-test the notebook at small sample sizes before the full run (as was done for `03`).
