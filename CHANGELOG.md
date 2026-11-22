# Changelog

Project history, newest first. Changes reach `main` through pull requests.

## 2026-11-22 · Documentation cleanup (branch `docs/readme-update`)
- README cut to a short overview; history and decisions moved here.
- `03_clustering.ipynb` introduction now states that complete linkage was adopted (average linkage was the original plan).
- Reports and docs tightened; no numbers changed.

## 2026-11-18 · PR #4 merged: visualizations and insights
- `b823812` (2026-11-17, @Chavanon-bit): `05_visualization_insights.ipynb` with five figures in `outputs/visualization_insights/`, four insights and an ethical reflection.

## 2026-11-11 · PR #3 merged: feature-engineering experiment
- `3642f2c`: `04_feature_engineering.ipynb` tested four cleaner feature sets under rules fixed in advance. None met the rules; **primary model kept**.
- `c013c62`: `reports/evaluation_summary.md` collects all evaluation scores.

## 2026-11-11 · PR #1 and #2 merged: clustering and results
- `aa7d7ba`: 10,000-child clustering, evaluation for k = 2–10, assignment, stability and sensitivity runs, profiles, exports to `outputs/`.
- `f66377d`: `reports/model_summary.md`.
- `a206ea1`: README shortened; challenges log moved to `docs/challenges.md`.
- `d9091d0`: `.gitignore`.
- Decisions:
  - **k = 4 not adopted.** The most stable alternative (neighborhood count) is driven by a sex split and keeps no neighborhood information.
  - **Neighborhood count and W2 not adopted.** Neither is more stable than the primary model at k = 3.

## 2026-11-10 · Primary model approved
- **Complete linkage** instead of average: average linkage put 99% or more of children in one cluster in all five samples.
- **Equal weights (W0) and k = 3:** best silhouette for k = 2–10, no cluster under 1%, acceptable stability, separates an older group.
- `9d10e1d`: README update.

## 2026-11-06 · Preprocessing fixes and clustering setup
- `40bbb6d`: child identifier, survey metadata, imputation flags, per-form sleep imputation, `sleep_below_minimum`.
- `7a30f5e`: `03_clustering.ipynb` Sections 0–6 (Gower implementation and validation, sampling, pilot), sleep and eye-care comparability review, first full README.

## 2026-11-04 · Project organized
- `c79b593`: preprocessing and EDA split into separate notebooks.

## 2026-11-03 · Initial commit
- `1225b72`.
