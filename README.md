# Sugarcane yield and quality prediction

Supervised learning on harvest records from **Ingenio Providencia** (Valle del Cauca, Colombia). The project models two mill outcomes on each harvested lot (*suerte*) and explores a variety-specific classification table.

| Outcome | Meaning | Notebook |
|---|---|---|
| **TCH** | Tonnes of cane per hectare (productivity) | `modelo_suertes.ipynb` |
| **%Sac.Caña** | Sucrose content of the cane (quality) | `modelo_suertes.ipynb` |
| **IPSA / CC01-1940** | Variety-level table for classification | `modelo_ipsa.ipynb` |

The unit of analysis in the harvest history is one lot harvested in a given month: `(Hacienda, Suerte, Periodo)`.

Open the notebooks in Google Colab (upload the Excel files in the Colab session; they are not on GitHub):

- Harvest history (regression): [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juanestebancg2806/sugarcane-yield-quality-prediction/blob/main/modelo_suertes.ipynb)
- IPSA CC01-1940 (classification): [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juanestebancg2806/sugarcane-yield-quality-prediction/blob/main/modelo_ipsa.ipynb)

## Repository layout

```
.
├── README.md
├── modelo_suertes.ipynb   # EDA, predictor selection, regression
├── modelo_ipsa.ipynb      # IPSA variety table (classification)
├── HISTORICO_SUERTES.xlsx # local only — not in git
└── BD_IPSA_1940.xlsx      # local only — not in git
```

## Data (not in this repository)

Mill spreadsheets are **gitignored**. Clone the repo, then place the Excel files next to the notebooks:

| File | Used by | Role |
|---|---|---|
| `HISTORICO_SUERTES.xlsx` | `modelo_suertes.ipynb` | ~21k harvests × 85 columns (2017–2024) |
| `BD_IPSA_1940.xlsx` | `modelo_ipsa.ipynb` | IPSA records for variety CC01-1940 |

Notebooks load them with `pd.read_excel(...)` from the working directory.

## `modelo_suertes.ipynb`

EDA and regression on the harvest history.

1. **Inventory and quality.** Completeness, identifiers vs agronomic measures, label cleanup.
2. **Predictor set.** Age of the current cycle, cut number, variety/soil/zone (rare levels grouped), distance to the mill, ripener dose, cycle and ripening rainfall, irrigation, harvest/burn type, crop system, tenure, seed destination, and year/month. Leakage from the same harvest (TCHM, TAH, lab sucrose, tonnes) is excluded. Undocumented dictionary fields are reconstructed from the data before they are dropped.
3. **Models.** Hold-out 80/20 *before* imputation. Train-only medians/modes and dummies. OLS (`statsmodels`) is the interpretable benchmark (signs, p-values). Ridge and Lasso are regularization checks; Random Forest is a non-linear performance comparison, not a substitute for OLS.

## `modelo_ipsa.ipynb`

Classification workflow on `BD_IPSA_1940.xlsx` (variety CC01-1940). Still in progress relative to the harvest-history notebook.

## Setup

Python **3.13+**. From this directory:

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install jupyter pandas numpy matplotlib seaborn scipy statsmodels scikit-learn openpyxl
jupyter notebook
```

Then open `modelo_suertes.ipynb` or `modelo_ipsa.ipynb` and run all cells.

## Design notes

- Targets are not imputed. TCH uses all 21,027 lots; sucrose uses the 20,578 rows with a measured value.
- Imputation rules come from the EDA (ripener NA → 0; irrigation count NA → 0 when volume is 0; soil/distance/tenure by zone), estimated on **train only**.
- OLS is the model used to read agronomic levers. Random Forest reports RMSE/MAE/R² only.
- Fertilization is almost unmeasured in this history (80–100% missing) and does not enter the linear models.
