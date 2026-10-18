# Sugarcane yield and quality prediction

Supervised learning on harvest records from **Ingenio Providencia** (Valle del Cauca, Colombia). The project has two separate studies on two different datasets:

| Study | Dataset | Targets | Task | Notebooks |
|---|---|---|---|---|
| Harvest history | `HISTORICO_SUERTES.xlsx` | TCH, %Sac.Caña | Regression | `01_eda_suertes.ipynb`, `02_modelos_regresion.ipynb`, `03_comparacion_modelos_suertes.ipynb` |
| IPSA CC01-1940 | `BD_IPSA_1940.xlsx` | TCH and sucrose classes (low / medium / high) | Classification | `01_eda_ipsa.ipynb`, `02_modelos_clasificacion.ipynb` |

- **TCH:** tonnes of cane per hectare (productivity).
- **%Sac.Caña / sucrose:** sucrose content of the cane (quality).

In both datasets the unit of analysis is one lot (*suerte*) harvested in a given month: `(Hacienda, Suerte, Periodo)`.

## Open in Google Colab

The Excel files are not on GitHub. In each Colab session, upload the Excel file and run the `01` notebook first: it writes the parquet file that the `02` and `03` notebooks read. Alternatively, upload the parquet file generated locally.

- Harvest history — EDA: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juanestebancg2806/sugarcane-yield-quality-prediction/blob/main/01_eda_suertes.ipynb)
- Harvest history — regression (first study): [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juanestebancg2806/sugarcane-yield-quality-prediction/blob/main/02_modelos_regresion.ipynb)
- Harvest history — model comparison (OLS, Ridge/Lasso, Random Forest, XGBoost): [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juanestebancg2806/sugarcane-yield-quality-prediction/blob/main/03_comparacion_modelos_suertes.ipynb)
- IPSA CC01-1940 — EDA: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juanestebancg2806/sugarcane-yield-quality-prediction/blob/main/01_eda_ipsa.ipynb)
- IPSA CC01-1940 — classification: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juanestebancg2806/sugarcane-yield-quality-prediction/blob/main/02_modelos_clasificacion.ipynb)

`03_comparacion_modelos_suertes.ipynb` runs long hyperparameter searches. Without the saved results (see [Saved search results](#saved-search-results)), a full run takes about 1 h 15 min on a MacBook Pro M3 with 8 GB of RAM, and longer on a free Colab instance.

## Repository layout

```
.
├── README.md
├── 01_eda_suertes.ipynb              # harvest history: inventory, data quality, EDA, predictors (steps 1–14)
├── 02_modelos_regresion.ipynb        # harvest history: hold-out, OLS, CV, Ridge/Lasso, RF, robustness (steps 15–24)
├── 03_comparacion_modelos_suertes.ipynb  # harvest history: farm-grouped split, OLS, Ridge/Lasso, RF, XGBoost, lot history, test, interpretability (sections 1–9)
├── 01_eda_ipsa.ipynb                 # IPSA CC01-1940: inventory, data quality, EDA, class definitions (sections 0–9)
├── 02_modelos_clasificacion.ipynb    # IPSA CC01-1940: partition, logistic regression, KNN, thresholds (sections 10–18)
├── Informe_Final_Industria_Azucarera.pdf   # final report
├── Tarea1_Contexto_del_negocio_APA.pdf     # task 1: business context
├── HISTORICO_SUERTES.xlsx            # local only — not in git
├── BD_IPSA_1940.xlsx                 # local only — not in git
├── datos/                            # local only — not in git
│   ├── suertes_limpio.parquet        # written by 01_eda_suertes, read by 02_modelos_regresion and 03_comparacion_modelos_suertes
│   └── ipsa_limpio.parquet           # written by 01_eda_ipsa, read by 02_modelos_clasificacion
└── resultados/                       # local only — not in git: saved searches and robustness checks of 03_comparacion_modelos_suertes
```

## Data (not in this repository)

The assignment does not allow sharing the data, so the mill spreadsheets and the `datos/` folder are **gitignored**. Clone the repo, then place the Excel files next to the notebooks:

| File | Used by | Content |
|---|---|---|
| `HISTORICO_SUERTES.xlsx` | `01_eda_suertes.ipynb` | 21,027 harvests × 85 columns (2017–2024) |
| `BD_IPSA_1940.xlsx` | `01_eda_ipsa.ipynb` | 2,187 harvests of variety CC01-1940, chemically ripened and mechanically harvested |

## Harvest history (`01_eda_suertes` + `02_modelos_regresion`)

1. **Inventory and data quality (`01`).** Completeness, disguised missing values, identifiers vs agronomic measures, label cleanup, units reconstructed for fields missing from the data dictionary, and outliers judged with business criteria.
2. **Predictor set (`01`).** Age of the current cycle, cut number, variety/soil/zone (rare levels grouped), distance to the mill, ripener dose and weeks, rainfall by cycle window and before harvest, irrigation, harvest and burn type, crop system, tenure, seed destination, and year/month. Leakage from the same harvest (TCHM, TAH, lab measurements, tonnes) is excluded.
3. **Models (`02`).** Hold-out 80/20 *before* imputation, with train-only medians, modes and dummies. OLS (`statsmodels`) is the interpretable benchmark (signs, p-values, assumptions). Ridge and Lasso are regularization checks. Random Forest is a non-linear performance comparison, not a substitute for OLS.
4. **Robustness (`02`, step 23).** Cross-validation grouped by farm (*hacienda*) and by harvest month, OLS with robust and clustered standard errors, a small Random Forest grid, and cross-validation with preprocessing refit inside a `Pipeline`. The row-wise hold-out shares lots between train and test, so grouped validation gives the expected performance on new farms or months, where the Random Forest advantage over OLS is much smaller.

## Harvest history: model comparison (`03_comparacion_modelos_suertes`)

This notebook rebuilds the regression study from the start, using the lessons of `02`:

1. **Preprocessing (section 2).** All imputation, rare-level grouping and encoding live in a custom transformer inside a `Pipeline`, so each fold only uses its own training data.
2. **Partition (section 3).** Hold-out 80/20 grouped by farm (no farm appears in both sets), plus five fixed cross-validation folds on the training set, also grouped by farm. All models share the same folds.
3. **Benchmarks (section 4).** Constant mean, OLS with farm-clustered standard errors (significance, sign and magnitude of the coefficients, assumptions checked with class tools), and Ridge/Lasso.
4. **Advanced models (section 5).** Random Forest and XGBoost, tuned with staged `GridSearchCV` and a review of the grid edges. Each model is also fitted with the **lot history**: TCH and sucrose of the lot's previous cut.
5. **Comparison (section 6).** Fold-by-fold paired comparison, a selection rule fixed in advance, and a robustness check that also groups by harvest year. The chosen model for both targets is **XGBoost with lot history**.
6. **Test set (section 7).** A single evaluation on the test set, with a farm-level cluster bootstrap for RMSE differences, actual vs predicted plots and error by subgroup.
7. **Interpretability and conclusions (sections 8–9).** Permutation importance, XGBoost gain, partial dependence and an agronomic summary, followed by proposed improvements and practical and ethical limitations.

Test results of the chosen model: RMSE 23.11 t/ha and R² 0.472 for TCH, and RMSE 0.837 points and R² 0.465 for %Sac.Caña. The multiple linear regression reaches R² 0.304 and 0.316.

### Saved search results

The hyperparameter searches, the robustness checks and the permutation importance are saved with `joblib` in `resultados/`, which is **gitignored**:

- `busqueda_{rf,xgb,rf_hist,xgb_hist}_{tch,sac}.joblib`
- `robustez_doble_agrupacion.joblib`
- `robustez_control_tamano.joblib`
- `importancia_permutacion.joblib`

When a file exists, the notebook loads it instead of recomputing; when it does not, the search runs and saves its result. Writes are atomic, so an interrupted run does not leave a corrupt file. Set `RECALCULAR_BUSQUEDAS = True` in section 5.1 to force all searches to run again. With the folder in place, a full run takes a few minutes.

## IPSA CC01-1940 (`01_eda_ipsa` + `02_modelos_clasificacion`)

1. **EDA (`01`).** Column meaning and units, missing and invalid values (rainfall zeros treated as missing), outliers, predictor–target relationships, cyclic encoding of the harvest month, and the class definitions.
2. **Classes.** Each target is split into low / medium / high with two criteria: the mill's thresholds (TCH < 125 and > 150 t/ha; sucrose < 12.2 % and ≥ 13.0 %) and training-set terciles as a reference.
3. **Partition (`02`).** Stratified train/test split grouped by harvest period (`StratifiedGroupKFold`), so that harvests from the same month never fall in both sets. Cross-validation uses the same grouping.
4. **Models (`02`).** Multinomial logistic regression (L1/L2, class weighting) and KNN, compared with a stratified random baseline. Pipeline with train-only imputation, capping and scaling. Metrics: macro F1, recall of the low class, Cohen's kappa and AUC.
5. **Interpretation and thresholds (`02`).** Coefficients read as odds ratios, analysis of the low-class alert threshold under different cost scenarios, and final model choice (logistic regression in the four target × class-definition cases).

## Setup

Python **3.13+**. With [uv](https://docs.astral.sh/uv/), from this directory:

```bash
uv venv
uv pip install jupyter pandas numpy matplotlib seaborn scipy statsmodels scikit-learn xgboost joblib openpyxl pyarrow
uv run jupyter notebook
```

Or with the standard library `venv` and pip:

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install jupyter pandas numpy matplotlib seaborn scipy statsmodels scikit-learn xgboost joblib openpyxl pyarrow
jupyter notebook
```

Run order:

- Harvest history: `01_eda_suertes.ipynb` → `02_modelos_regresion.ipynb` and/or `03_comparacion_modelos_suertes.ipynb`. Both modelling notebooks read the parquet file written by `01`.
- IPSA CC01-1940: `01_eda_ipsa.ipynb` → `02_modelos_clasificacion.ipynb`.

Reported results:

- `02` notebooks: Python 3.13.9, pandas 3.0.5, scikit-learn 1.9.1, statsmodels 0.15.0 and matplotlib 3.11.2.
- `03_comparacion_modelos_suertes.ipynb`: Python 3.13, pandas 3.0.6, scikit-learn 1.9.1, statsmodels 0.15.0 and xgboost 3.4.1.

Other versions can change the third decimal of some cross-validation metrics.

## Design notes

- Targets are never imputed. In the harvest history, TCH uses 20,672 lots (355 mill lots with TCH < 60 t/ha are excluded as probable recording errors) and sucrose uses the 20,578 rows with a measured value.
- Imputation rules come from the EDA and are estimated on **train only**: ripener dose NA → 0; irrigation count NA → 0 when the volume is 0; missing soil → explicit `SIN DATO` category; distance, tenure and crop system by zone.
- Rainfall zeros are mostly missing measurements. The regression keeps them with a `lluvia_sin_dato` flag; the classification imputes them with the training median of the harvest month, because KNN would treat a zero as a real measurement.
- In `03`, the lot history only uses earlier cuts of the same lot that are at least 6 months before the current harvest, so a record never sees its own outcome.
- In `02`, OLS is the model used to read agronomic levers and Random Forest reports RMSE/MAE/R² only. In `03`, the tree models are also interpreted, with permutation importance, XGBoost gain and partial dependence.
- Fertilization is almost unmeasured in the harvest history (80–100 % missing) and does not enter the models.