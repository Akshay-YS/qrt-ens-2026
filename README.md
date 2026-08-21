# QRT Trust-or-Short

Modeling work for the QRT / ENS Data Challenge (Challenge 167): predicting the
sign of a financial return (`TARGET > 0`) from anonymized return, volume, and
allocation features. The repo contains two notebooks covering feature
exploration/engineering and the training pipeline.

## Notebooks

### `Feature_engineering_QRT.ipynb`
Exploratory analysis and feature engineering, run in Google Colab against
Drive-mounted data.

- **Allocation-level analysis** — distribution of row counts per allocation
  in train vs. test, and identification of small/underrepresented
  allocations.
- **Return distribution checks** — train vs. test comparison of mean/std of
  returns (`RET_1`–`RET_20`), including KS-statistic anomaly detection
  per allocation against the global distribution.
- **Target analysis** — per-allocation target statistics (mean, volatility,
  skew, kurtosis) and win-rate (`TARGET > 0` frequency) profiling, used to
  bucket allocations into behavioral regimes (strong short / noise / strong
  long).
- **Prediction diagnostics** — calibration curves and confidence analysis on
  out-of-fold predictions produced by the training notebook.
- **Feature grouping** — hierarchical clustering of engineered features by
  correlation (Spearman/Pearson/Kendall) to `TARGET_SIGN`, with dendrogram
  and heatmap visualization, used to select a diverse, low-redundancy
  feature subset for modeling.

### `QRT_Trust_or_short.ipynb`
The training pipeline: a configurable ensemble of classifiers with a
logistic-regression meta-model, designed to run on Kaggle.

- **Configurable feature blocks** — feature engineering is split into
  blocks that can be toggled per model via `FEATURE_BLOCKS_TO_USE`.
- **Models** — LightGBM and CatBoost classifiers (`MODEL_SPECS`), with GPU
  auto-detection.
- **Cross-validation** — date-based K-fold (`make_date_kfolds`) so folds
  split on unique timestamps (`TS`) rather than rows, avoiding leakage
  across the same date.
- **Multi-seed runs** — each model is trained across several CV/model seed
  configurations (`SEED_CONFIGS`) and averaged, then stacked into a
  logistic-regression meta-model.
- **Diagnostics output** — out-of-fold predictions, error-type breakdown
  (TP/FP/TN/FN), and per-model correlation (Pearson/Spearman) tables are
  saved for downstream analysis in the feature-engineering notebook.
- **Submission** — writes a thresholded (`>0.5`) prediction CSV in the
  competition's `ROW_ID, prediction` format.

## Suggested workflow

1. Run `Feature_engineering_QRT.ipynb` first to explore the data and
   generate engineered/diagnostic feature sets.
2. Run `QRT_Trust_or_short.ipynb` to train the ensemble and produce a
   submission file.
3. Feed the training notebook's saved OOF/diagnostic outputs back into the
   feature-engineering notebook for calibration and error analysis.

## Requirements

```
pandas
numpy
scikit-learn
lightgbm
catboost
scipy
matplotlib
seaborn
plotly
```

`Feature_engineering_QRT.ipynb` assumes a Google Colab environment (Drive
mount for data paths); `QRT_Trust_or_short.ipynb` assumes Kaggle-style input
paths (`/kaggle/input/...`). Update the path variables near the top of each
notebook for your environment.

## Data

Expects `train.csv` / `test.csv` from the QRT/ENS Challenge 167 dataset, with
columns including `TS`, `ALLOCATION`, `RET_1`–`RET_20`,
`SIGNED_VOLUME_1`–`SIGNED_VOLUME_20`, `TARGET`, and `ROW_ID`. Data is not
included in this repo.
