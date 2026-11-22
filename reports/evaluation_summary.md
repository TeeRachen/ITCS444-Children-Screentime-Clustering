# Model Evaluation Summary

All evaluation scores for the primary model (Gower distance, complete linkage, W0, k = 3) in one table. Separation is weak, stability is moderate, and the partition changes with weighting. All numbers come from `outputs/` and [`feature_engineering.md`](feature_engineering.md); nothing here is new analysis.

## Why these metrics?

Clustering has **no true labels** (screen time is a feature, not a target), so accuracy and confusion matrices do not apply.

- **Train-test equivalent:** five disjoint stratified samples of 10,000. Each is clustered, the other children are assigned (the "held-out" side), and the five full-data labelings are compared by Adjusted Rand Index (ARI).
- **Not used:** Davies-Bouldin and Calinski-Harabasz (they assume Euclidean geometry, which Gower does not give) and k-fold cross-validation (meaningless without labels).

## What are the scores? (k = 3 unless stated)

| Aspect | Measure | Result | Reading |
|---|---|---|---|
| Cluster quality | Mean silhouette, part 0 | 0.208 (k = 2: 0.199; k = 4: 0.122) | Weak structure (below 0.25) |
| | Silhouette across the five parts | 0.099 to 0.208 | Varies with the sample |
| | Cophenetic correlation, part 0 | 0.48 (average linkage 0.64) | Moderate fit of tree to distances |
| | Smallest cluster, any part | 271 of 10,000 (2.7%) | No cluster under 1% |
| Stability | Pairwise ARI, 10 pairs of parts | mean 0.525, range 0.302 to 0.805 (k = 2: 0.558; k = 4: 0.350) | Moderate; falls with larger k |
| Assignment | Margin below 0.005 / 0.01 / 0.02 | 1.9% / 3.3% / 5.9% of assigned children | Few ambiguous assignments; no ties |
| | Leave-one-out consistency | 89.2% overall; 67.7% in cluster 3 | Cluster 3 least clear-cut |
| Robustness | ARI vs primary: W1 / W2 / neighborhood count | 0.082 / 0.042 / 0.044 | Far below the stability band |
| | ARI vs primary: all binaries symmetric / with `eyedoctor` / without sleep / complete cases | 0.277 / 0.354 / 0.481 / 0.372 | Mostly inside or near the band |
| Feature sets | Mean pairwise ARI, candidates A / B / C / D (need 0.575) | 0.233 / 0.354 / 0.281 / 0.493 | None beat the primary (0.525) |

## How should the scores be read?

The clusters are a **description of this survey sample**, not natural groups. Cluster 3 (older, high parent-reported treatment and medication) is the most distinct but the least stable under reassignment. The model was kept because no tested alternative was more stable without tiny clusters or a single-feature split.

**Files:** `outputs/evaluation_results.csv`, `per_cluster_silhouette.csv`, `stability_sensitivity.csv`, `cluster_assignments.csv`, `model_config.json`; `outputs/feature_engineering/`.
