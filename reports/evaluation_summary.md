# Model evaluation summary

Primary model: complete linkage, equal weights (W0), k = 3, Gower distance. All numbers come from `outputs/` and `reports/feature_engineering.md`; nothing here is new analysis.

## Why these metrics

The course asks for evaluation metrics suited to the technique, and for cross-validation or train-test splits as needed. Clustering has **no true labels** (screen time is a clustering feature, not a target), so supervised metrics such as accuracy and confusion matrices do not apply. The evaluation uses internal quality measures, a stability check that plays the role of repeated train-test splits, and robustness checks.

- **Train-test equivalent:** five disjoint stratified samples of 10,000 children. Each is clustered, and the other children are assigned to the nearest cluster (the "held-out" side). The five full-data labelings are compared with the Adjusted Rand Index (ARI).
- **Not used:** Davies-Bouldin and Calinski-Harabasz (they assume Euclidean geometry, which Gower distance does not give) and k-fold cross-validation (meaningless without labels).

## Summary table (k = 3 unless stated)

| Aspect | Measure | Result | Reading |
|---|---|---|---|
| Cluster quality | Mean silhouette, part 0 | 0.208 (k = 2: 0.199; k = 4: 0.122) | Weak structure (below 0.25) |
| | Silhouette across the five parts | 0.099 to 0.208 | Varies noticeably with the sample |
| | Cophenetic correlation, part 0 | 0.48 (average linkage 0.64) | Tree reproduces the distances only moderately |
| | Smallest cluster, any part | 271 of 10,000 (2.7%) | No cluster under 1% |
| Stability | Pairwise ARI, 10 pairs of parts | mean 0.525, range 0.302 to 0.805 (k = 2: 0.558; k = 4: 0.350) | Moderate; falls with larger k |
| Assignment | Margin below 0.005 / 0.01 / 0.02 | 1.9% / 3.3% / 5.9% of assigned children | Few ambiguous assignments; no exact ties |
| | Leave-one-out consistency | 89.2% overall; 67.7% in cluster 3 | Cluster 3 is the least clear-cut |
| Robustness | ARI vs primary: W1 / W2 / neighborhood count | 0.082 / 0.042 / 0.044 | Well below the stability band: sensitive to weighting and coding |
| | ARI vs primary: all binaries symmetric / with `eyedoctor` / without sleep / complete cases | 0.277 / 0.354 / 0.481 / 0.372 | Mostly inside or near the band |
| Feature sets | Mean pairwise ARI, candidates A / B / C / D (need 0.575) | 0.233 / 0.354 / 0.281 / 0.493 | None better than the primary (0.525); see `reports/feature_engineering.md` |

## Reading the scores

The clusters are a **description of this survey sample**, not natural groups: separation is weak, stability is moderate, and the partition changes with the weighting. Cluster 3 (older, with high parent-reported mental-health treatment and medication use) is the most distinct but also the least stable under reassignment. The model was kept because no tested alternative was more stable without creating tiny clusters or a split driven by a single feature.

## Where the numbers live

`outputs/evaluation_results.csv`, `per_cluster_silhouette.csv`, `stability_sensitivity.csv`, `cluster_assignments.csv`, `model_config.json`; `outputs/feature_engineering/`.
