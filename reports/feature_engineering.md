# Feature-Engineering Experiment: Findings

Four cleaner feature sets were tested against the primary model under rules fixed in advance. **None met the rules, so the primary model was kept** (decided by the team on 2026-11-11).

**Plan:** [`docs/feature_engineering_plan.md`](../docs/feature_engineering_plan.md) · **Notebook:** `04_feature_engineering.ipynb` · **Results:** `outputs/feature_engineering/` (CSV only, no distance matrices). All results are associations in the unweighted 2024 NSCH sample; medication use is parent-reported, not a diagnosis.

## What was the result?

- **Stability:** no candidate reaches mean pairwise ARI ≥ 0.575. The best, D, has 0.493; the primary has 0.525.
- **Weighting:** none is less weighting-sensitive than its own replicate band allows.
- **Tiny clusters:** C and D produce clusters under 1% in some parts.
- **Single-feature splits:** A, B and C each have a feature that almost defines the split (Cramér's V ≥ 0.8). The primary fails this rule too, so the cleanup moved the problem rather than fixing it.

## What did the diagnostics show? (no clustering)

- **Strongest associations:** `k4q22_r` × `k4q23` (V 0.50); the three neighborhood items with each other (V 0.40–0.48); age × screen time (Spearman 0.50, a developmental pattern, not redundancy). All others V < 0.3.
- **Share of total Gower distance** (part 0, W0): `family_r` 11.2%, `sc_sex` 11.1%, `k10q13` 11.1%, `k10q14` 9.7%, `sleep_below_minimum` 9.4%, `screentime` 8.4%, `k10q12` 7.9%, age 7.8%, `k4q22_r` 6.3%, `k6q71_r` 5.6%, `k4q23` 3.9%, `foodsit` 3.6%, `higrade` 3.3%, `blindness` 0.6%.
- **By domain:** environment 35.6% (neighborhood items alone 28.7%), general 30.2%, lifestyle 23.4%, health 10.8%. Asymmetric binaries carry little distance because most pairs share "No".
- **Mental-health items** match the planned counts: none 41,253; unmet need only 978; treatment only 3,789; medication without treatment 1,924; both 3,431.
- **No extra candidate** was added (reasons in the notebook, written before clustering).

## How did each candidate score? (k = 3, complete linkage, W0)

| | Primary (ref.) | A no sex (13) | B merged mental health (13) | C combined (10) | D C without sleep (9) |
|---|---|---|---|---|---|
| R1 mean pairwise ARI (need ≥ 0.575) | 0.525 | 0.233 | 0.354 | 0.281 | 0.493 |
| (min / max) | 0.302 / 0.805 | 0.071 / 0.472 | 0.209 / 0.539 | 0.139 / 0.677 | 0.190 / 0.890 |
| R2 ARI W0 vs W1 (need ≥ own band min) | 0.082 (min 0.302) | 0.021 (0.071) | 0.009 (0.209) | 0.121 (0.139) | 0.085 (0.190) |
| R3 smallest cluster in any part (need ≥ 100) | 271 | 193 | 907 | **37** | **51** |
| R4 largest Cramér's V, part 0 (need < 0.8) | 0.84 (`k10q14`) | 0.86 (`k10q13`) | 0.86 (`k10q13`) | 0.93 (`sleep_below_minimum`) | 0.52 (`mh_care`) |
| R1 / R2 / R3 / R4 met? | no / no / yes / no | no / no / yes / no | no / no / yes / no | no / no / **no** / no | no / no / **no** / yes |
| Verdict | reference | not adopted | not adopted | not adopted | not adopted |

- **R2:** C is close (0.121 vs 0.139), but its band is very low, so this reflects weak stability, not robustness.
- **R3:** C has a 37-child cluster (part 1); D has 61 (part 1) and 51 (part 4).
- **R4** is judged on part 0, as fixed in advance. Across all parts the highest V is 0.84 (primary), 0.87 (A), 0.86 (B), 0.93 (C), 0.95 (D, part 2), so D would fail if all parts counted.
- **Primary re-run check:** its k = 3 labeling equals `outputs/cluster_assignments.csv` exactly (ARI 1.0), and its stability and W1 numbers match notebook 03.

## What did the candidates' clusters look like? (part 0, all children)

| Candidate | What drives the split |
|---|---|
| A, no sex | Amenities (recreation center 79–95% in two clusters, 0.4% in the third) plus sleep and age. The high-treatment group (30% treatment, 21% medication) blends with older age. |
| B, merged | Sex (one cluster 95% female, one 7%) plus amenities. No mental-health cluster at k = 3 (treatment 12–18% everywhere). |
| C, combined | Sleep below minimum (99.8% in one cluster, 2–5% elsewhere) and age (mean 7.0 / 10.5 / 13.3). |
| D, no sleep | Age, screen time and mental-health care. Cluster 3 (2,286 children): 64% treatment, 67% medication, 53% screen time 4+ h. Most stable candidate, but tiny clusters in some parts. |

## What else was observed? (informational, not decision rules)

- **Silhouettes** (own distance, not comparable across candidates), k = 3: primary 0.208, A 0.111, B 0.185, C 0.126, D 0.121.
- **ARI vs primary** (part 0, k = 3): A 0.158, B 0.446, C 0.018, D 0.043.
- **Under W1** silhouettes rise (C 0.313, B 0.289), but labelings change almost completely (ARI vs W0 0.01–0.12).
- **k = 2 mean ARI:** primary 0.558; A–D 0.100 / 0.322 / 0.226 / 0.579.
- **Assignment margins below 0.01** (k = 3, mean over parts): primary 7.7%, A 11.3%, B 7.1%, C 9.7%, D 5.1%.

## Method notes

- Same seed (42) and the same five disjoint parts as notebook 03 (verified), complete linkage, W0 Gower, assignment by mean distance. W1 runs on part 0 only.
- **Tied merge heights:** one informational cut (D, part 0, k = 5) returned 4 clusters; the notebook undid the last k − 1 merges instead (flag `tie_fallback_cut`). No decision number is affected.
- Cramér's V is uncorrected, with every feature categorical (age as 18 levels). Fixed before the run.

## Conclusion

The experiment confirms the known limits: clusters are weak, mainly reflect which binary items are included (amenities, sleep, sex), and change with weighting. One pattern recurs: an older group with more screen time and higher parent-reported treatment and medication. It is sharp in the primary (78% treatment, 62% medication, 3,184 children) and in D, diluted in A and C, and absent in B at k = 3.
