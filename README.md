# Next-Day AQI Forecasting for Four Indian Cities

ES 678 course project (IIT Gandhinagar), Manav Jain (22110140).

**Question.** Using only air-quality data available at the end of day *t*, how accurately can ridge regression, random forest and CatBoost forecast the AQI of day *t+1* for Delhi, Bengaluru, Kolkata and Hyderabad, and how much do they improve on persistence (tomorrow's AQI = today's)?

**Main result.** On the 2020 test period the models lower MAE by 15–17% relative to persistence (ridge 13.84 vs. 16.75 AQI points), but only before the COVID-19 lockdown; during it no model beats persistence. An ablation shows that common evaluation shortcuts inflate the random forest's apparent skill over persistence from 15% to 46%.

## Repository contents

| File | What it does |
|---|---|
| `notebooks/main_pipeline.ipynb` | Data audit, cleaning, features, chronological split, baselines, model tuning, final test, block-bootstrap CIs; saves `features.pkl` and `state.pkl` |
| `notebooks/ablation.ipynb` | Ablation of evaluation design (S0–S5) and the paper's ablation and Delhi figures |
| `figures/` | Figures used in the paper |
| `requirements.txt` | Python packages |

## Data access

The data are **not** included in this repository. Download `city_day.csv` from the public Kaggle dataset *Air Quality Data in India (2015–2020)* by Vopani:
<https://www.kaggle.com/datasets/rohanrao/air-quality-data-in-india>
(free Kaggle account required). Only `city_day.csv` is used. No credentials or API keys are needed or stored here.

## How to reproduce

The notebooks were run on Google Colab (Python 3).

1. Open `notebooks/main_pipeline.ipynb` in Colab. Upload `city_day.csv` when the upload cell asks, or put it in Google Drive and set `DATA_PATH`.
2. The notebook writes intermediate files to `/content/drive/MyDrive/aqi_project/`. Change that path if you use a different folder.
3. Run all cells in order. Cells install `catboost` and `imbalanced-learn` with `pip`.
4. Then run `notebooks/02_ablation.ipynb`, which loads `features.pkl` and `state.pkl` from the same folder.

Approximate run time: under 10 minutes for the main notebook and 5–10 minutes for the ablation.

## Key settings

- **Cities:** Delhi, Bengaluru, Kolkata, Hyderabad; one pooled model with city indicators.
- **Cleaning:** Xylene dropped (mostly missing); zeros in NOx, CO, Benzene and Toluene treated as missing.
- **Target:** AQI on day *t+1* (rows without it dropped: 6,542 rows remain).
- **Features (94):** AQI and 11 pollutants at lags 0–6; 3- and 7-day rolling means of AQI and PM2.5; month and day of week of the target day; city one-hot. All shifts are within city.
- **Split (by target date):** train 2015–2018, validation 2019, test 1 Jan–1 Jul 2020. Final models are refit on 2015–2019 and tested once.
- **Leakage control:** median imputation (and scaling for ridge) fitted inside pipelines on training rows only; CatBoost uses its native missing-value handling; tuning uses validation only.
- **Hyperparameters:** ridge alpha = 0.1; random forest 300 trees, min_samples_leaf = 20, max_features = 1.0; CatBoost depth 8, learning rate 0.05, 962 iterations.
- **Seeds:** 42 (models), 0–2 (ablation), 0 (bootstrap).
- **Lockdown split of the test set:** 25 March 2020.

## References for external code and data

- Dataset: Vopani, *Air Quality Data in India (2015–2020)*, Kaggle, 2020 (from CPCB records).
- scikit-learn (Pedregosa et al., JMLR 2011); CatBoost (Prokhorenkova et al., NeurIPS 2018); imbalanced-learn (Lemaître et al., JMLR 2017); pandas, NumPy, Matplotlib.
- Full references are in the paper.
