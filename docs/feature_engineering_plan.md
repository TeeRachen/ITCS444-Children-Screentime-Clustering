# Feature-Engineering Experiment: Fixed Plan

The plan for `04_feature_engineering.ipynb`, **fixed on 2026-11-11 before anything was run**. Candidates and decision rules were not changed after seeing results. Findings: [`../reports/feature_engineering.md`](../reports/feature_engineering.md).

## What was the goal?

The primary model (complete linkage, W0, k = 3) gives weak clusters (silhouette 0.208) that change with weighting and with how the three correlated neighborhood items are coded (ARI vs primary 0.04–0.08). The experiment tests whether a cleaner feature set is more stable and less sensitive. The primary model and its outputs stay unchanged.

## Step 1: diagnostics (no clustering)

- Association matrix of the 14 features (Cramér's V for categorical, Spearman for ordinal).
- Each feature's share of the total pairwise Gower distance (part 0).
- Prevalence of each binary feature.
- Cross-table of the mental-health items `k4q22_r` (1 received, 2 needed not received, 3 not needed) × `k4q23` (1 medication, 2 none).
- At most **one** extra candidate may be added, written into the notebook before any clustering.

## Step 2: which candidates? (W0, complete linkage)

| | Candidate | Change from primary | Features |
|---|---|---|---|
| A | No sex | `sc_sex` moves to profiling only | 13 |
| B | Merged mental health | `k4q22_r` + `k4q23` become one nominal `mh_care`: 1 none, 2 unmet need only, 3 treatment only, 4 medication without treatment, 5 both | 13 |
| C | Combined cleanup | A + B + `k10q12`/`k10q13`/`k10q14` replaced by an ordinal `amenity_count` (0–3) | 10 |
| D | C without sleep | C minus `sleep_below_minimum` (overlaps with age) | 9 |

## Step 3: how was each candidate judged?

Each candidate runs on all five parts (cluster 10,000, assign 41,375), k = 2–6, seed 42, the same parts as notebook 03. It also runs with W1 on part 0 and is compared with its own W0 labeling.

**Decision rules, at k = 3 only:**

1. **More stable:** mean pairwise ARI ≥ 0.575 (primary 0.525).
2. **Less weighting-sensitive:** W0-vs-W1 ARI at or above its own replicate band minimum (primary 0.082).
3. **No tiny clusters:** none under 1% of the sample in any part.
4. **No single-feature split:** no feature with Cramér's V ≥ 0.8 against the cluster labels.
5. Silhouettes are reported but **not compared** across candidates.
6. Other k values are informational; k is not chosen per candidate.

A candidate meeting rules 1–4 is "worth adopting"; otherwise it is reported as tested. The team decides whether to change the primary model.
