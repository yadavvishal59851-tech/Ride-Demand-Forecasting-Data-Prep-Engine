# Summary Report: Ride Demand Data Prep

**Goal.** Combine riders (300), trips (2,000) and city zones (10) into one clean, model-ready table for demand forecasting and surge prediction.

## Theory in brief
- **Imputation:** mean for numeric and most-frequent for categorical gaps (simple, but shrinks variance). KNN (k=5) for distance, duration and fare, because they are correlated; columns are standardised first so none dominates.
- **Outliers:** Z-score (|z|>3) for fare and distance, IQR (1.5x) for duration, which is skew-friendly. Winsorization caps extreme values instead of deleting rows, so no data is lost.
- **Transforms:** log and square-root reduce right skew. Label encoding for gender, one-hot for payment mode / zone (no false order), ordinal encoding for Low < Medium < High traffic.
- **Scaling:** StandardScaler (mean 0, sd 1) and MinMaxScaler (0-1). When modelling, fit scalers on training data only to avoid leakage.

## Observations
| Metric | Before | After |
|---|---|---|
| Trip rows | 2,040 | 1,976 (duplicates, negative-fare and zero-distance rows removed) |
| Missing values | 204 | 0 |
| Outliers (fare / distance / duration) | 17 / 2 / 16 | 0 / 0 / 0 |
| Engineered or transformed features | 0 | 13 |

- Log transforms **over-correct** (fare skew +0.80 becomes -1.49), since the raw skew was mild. Square-root on duration helps (+0.44 to -0.39).
- The provided `surge_flag` flags ~51% of trips and matches the fare/km rule (top 25%, threshold ~20.5) only 48% of the time, which is chance level. Both are kept (`surge_flag_raw`, `surge_flag`).

## Limitations
1. `hour` is **synthetic** (the data only has dates), so peak-hour and demand-by-hour results only show that the pipeline works.
2. The original files were already clean, so missing values, duplicates, mixed date formats and bad rows were **added on purpose** (random seed 42) to demonstrate cleaning. The before/after numbers describe that test.
3. Rider averages use all trips; for a real forecast they should use past trips only.
