# Children's Screen-Time Clustering

Groups 51,375 U.S. children from the 2024 National Survey of Children's Health by health, lifestyle and environment, then compares screen time across the groups. Run the notebooks in order: 01 → 02 → 03 → 05.

**Research question:** what health, lifestyle and environmental patterns are associated with children's screen-time levels? Results are **associations only**, never causes.

**Course:** ITCS444 Data Mining · **Instructor:** Aj. Charnyote Pluempitiwiriyawej

## What did we find?

Gower distance, complete linkage, k = 3; 10,000 children clustered, the rest assigned to the nearest cluster.

| | Cluster 1 | Cluster 2 | Cluster 3 |
|---|---|---|---|
| Children | 33,714 (65.6%) | 14,477 (28.2%) | 3,184 (6.2%) |
| Mean age | 9.6 | 8.9 | 13.2 |
| Park / rec. center / library nearby (%) | 96 / 69 / 96 | 37 / 6 / 12 | 46 / 10 / 22 |
| Mental-health treatment (%, parent-reported) | 13 | 2 | 78 |
| Screen time 4+ h (%) | 22.5 | 21.4 | 51.3 |

**The structure is weak** (silhouette 0.208, replicate ARI 0.525) and changes with feature weighting. Read the clusters as a description of this survey sample, not as natural groups.

## How to run it

1. Clone the repository:
   ```
   git clone https://github.com/TeeRachen/ITCS444-Children-Screentime-Clustering.git
   ```
2. Use **Python 3.12** with pandas 2.2, NumPy 1.26, SciPy 1.13, scikit-learn 1.5, matplotlib 3.9 and psutil (the project used Anaconda `base`).
3. Run the notebooks in order. `03_clustering.ipynb` takes about 35 minutes and needs **about 1.5 GiB of free RAM**.

## Where things are

| Path | Content |
|---|---|
| `01_preprocessing.ipynb` · `02_eda.ipynb` | Cleaning, imputation, exploratory analysis |
| `03_clustering.ipynb` | Gower distance, clustering, evaluation, sensitivity runs, exports |
| `04_feature_engineering.ipynb` | Optional experiment (about 55 min); primary model unchanged |
| `05_visualization_insights.ipynb` | Figures, insights, ethical reflection |
| `datasets/` | Raw NSCH `.dta` and `preprocessed_data.csv` |
| `outputs/` | Cluster labels, profiles, evaluation tables, figures |
| `reports/` | [Model summary](reports/model_summary.md) · [Evaluation](reports/evaluation_summary.md) · [Feature engineering](reports/feature_engineering.md) |
| `docs/` | [Challenges and failed attempts](docs/challenges.md) · [Experiment plan](docs/feature_engineering_plan.md) |

## Limits

- Unweighted clustering of an oversampled survey: no generalization to all U.S. children.
- Cluster 3 is the oldest, and age is a clustering feature, so its higher screen time is not separable from age.
- Data are cross-sectional and parent-reported; medication use is not a diagnosis.

## Team, data and history

| Role | Member |
|---|---|
| Model implementation and evaluation | [@TeeRachen](https://github.com/TeeRachen) |
| Visualization, insights, ethical reflection | [@Chavanon-bit](https://github.com/Chavanon-bit) |

Data: U.S. Census Bureau, *2024 National Survey of Children's Health, Topical Public Use File* ([source](https://www.census.gov/programs-surveys/nsch/data/datasets/nsch2024.html)), analyzed in aggregate only. The repository has no license, and nothing here applies to the NSCH data. Project history: [CHANGELOG.md](CHANGELOG.md).
