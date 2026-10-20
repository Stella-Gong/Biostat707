# BIOSTAT 707 — ICU mortality project

PhysioNet Challenge 2012 set-a: cohort description (Checkpoint 1) and predictive modeling (Checkpoint 2).

## Data and environment

Install Pixi and place the supplied, extracted data in the repository:

```text
data/
├── set-a/           # 4,000 ICU record files, e.g. 132539.txt
└── Outcomes-a.txt
```

Raw data are not committed. Checkpoint 1 reads `data/`, checks the original value ranges, and then changes impossible values and −1 codes to NaN before creating `output/set-a_long.csv` and `output/set-a_wide.csv`. The cleaning code is written directly in Section 4 of the notebook.

Checkpoint 2 reads only `output/set-a_wide.csv` and uses its cleaned summaries directly.

All three models fill missing numeric predictors with the training-fold mean of each feature; Gender and ICUType use the training-fold mode. Counts and missingness indicators are retained. All three models standardize inputs after imputation and encoding using training-fold statistics. It does not repeat raw cleaning or aggregation. Both checkpoints write under `output/`.

## Run and read the reports

From the repository directory:

```bash
pixi run --locked checkpoint2
```

The `checkpoint2` Pixi task depends on `checkpoint1`. This single command first executes Checkpoint 1 to clean raw measurements and generate its long/wide tables, then executes Checkpoint 2. The dependency runs Checkpoint 1 each time this command is invoked. To run Checkpoint 1 alone, use `pixi run --locked checkpoint1`.

The rendered reports are:

- `output/checkpoint1.html`: data checks, cohort characteristics, outcomes, and missingness.
- `output/checkpoint2.html`: five-fold cross-validation (without nesting), ridge logistic regression, restricted cubic spline logistic regression, random forest, SAPS-I/SOFA benchmarks, interpretation, and final model choice.


On this laptop, the latest locked run took about 57 seconds for Checkpoint 1 → Checkpoint 2, including about 37 seconds for the Checkpoint 2 analysis. Checkpoint 1 plus notebook startup/export accounts for about 20 seconds. This excludes environment downloads; runtime varies by machine.

## Checkpoint 2 implementation

- `checkpoint2.ipynb` is the modeling report and analysis entry point. The committed notebook has cell outputs removed by nbstripout; results are in the HTML report.
- All preprocessing, feature selection, splines, and model definitions are written in `checkpoint2.ipynb`.
- There are 222 predictors from the cleaned wide table. IDs, outcomes, benchmark scores, and first-value summaries are excluded; the MechVent summaries are replaced by a documentation indicator.
- Fixed seed: 707. Five shared stratified folds and four tuning configurations per model. Hyperparameters minimize mean CV log loss. The same folds are reused for evaluation, so reported scores may be optimistic.
- Numeric feature means, categorical modes, spline knots, encoding, and scaling are fitted inside the cross-validation pipelines. No validation stays contribute to those estimates.
- Model 1 is ridge logistic regression with all 222 candidate inputs. Model 2 uses the top 15 original inputs by absolute standardized ridge coefficient (maximum across dummy coefficients for categorical inputs). Its ridge selector uses the C selected for Model 1 and fits within every training split, without additional CV. Continuous selected measurements use restricted cubic splines; counts enter linearly. The final 15 inputs match the full-data Model 1 ranking.
- Model 3 is a random forest with 300 trees. The four settings combine maximum depth 10 or unlimited with minimum leaf size 1 or 5; max_features is sqrt. Each model section separately lists its tuning grid and per-fold AUROC with mean and SD.
- Final choice: keep models within 0.01 of the best mean fold AUROC, then prefer ridge, splines and random forest in that order. Both hyperparameter and model-family selection can make the winning CV estimate optimistic.

Important generated files:

```text
output/checkpoint2.html
output/cp2_final_model.joblib       # Fitted preprocessing and final model
output/cp2_oof_predictions.csv      # IDs, outcomes, folds, 3 model risks, 2 benchmark scores
output/cp2_metadata.json            # Input order, selected settings, package versions, timing
output/cp2_comparison.csv           # Five approaches: per-fold AUROC, mean, SD
output/cp2_fold_metrics.csv         # Model AUROC, log loss, Brier by fold
output/cp2_tuning.csv               # Five-fold candidate scores and selected settings
output/cp2_ridge_importance.csv     # Full ridge ranking of original inputs
output/cp2_spline_selected_features.csv # Final 15 inputs and fold selection counts
output/cp2_spline_fold_features.csv # Separate top-15 list for each training fold
output/cp2_modeling_table.csv       # Cleaned 222 inputs and outcome; NaN retained before fold imputation
output/cp2_imputation_means.csv     # Numeric feature means from the full-data ridge fit
output/cp2_fold_imputation_means.csv # Numeric feature means from each training ridge fit
output/raw_cleaning_summary.csv    # CP1: separate original-missing/extreme counts, retained ranges, descriptive means
output/raw_cleaning_removed.csv    # CP1: masked original records, IDs, times and MissingRaw/Impossible flags
```

The output directory also contains the plotted figures, spline effects, and full ridge coefficients.

## Git submission

Commit notebook sources, README, `pixi.toml`, `pixi.lock`, `.gitignore`, `.gitattributes`, and the two HTML reports. Raw data, the local environment and generated supporting files stay local.

In each clone, enable notebook output stripping before committing:

```bash
pixi run nbstripout --install --attributes .gitattributes
```

Before submission, clone the repository into a fresh directory, provide `data/`, and run only `pixi run --locked checkpoint2`.
