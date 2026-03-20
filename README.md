# Predicting AMR Patterns in Kazakhstan Using Zero-Shot Machine Learning

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![Kaggle](https://img.shields.io/badge/Notebook-Kaggle-20BEFF.svg)](https://www.kaggle.com/code/uaisamangeldi/amr-kazakhstan-zeroshot-v2)

## The Problem

Kazakhstan does not have a nationally integrated antimicrobial resistance (AMR) surveillance system. This means there is no local clinical data to train a prediction model on — which is exactly the situation where most machine learning approaches break down.

We wanted to answer a simple question: **can a model trained entirely on data from other countries make useful resistance predictions for Kazakhstan?**

## What We Built

A zero-shot machine learning framework that trains on three global AMR surveillance databases — ATLAS, SIDERO-WT, and KEYSTONE — covering 12.4 million clinical isolate records from 85 countries between 2004 and 2023. Kazakhstan is completely excluded from training.

Instead of using country name as a feature (which would make generalization to unseen countries impossible), the model represents each country through its macroeconomic profile: GDP per capita, health expenditure as a percentage of GDP, and population density. These act as proxies for a country's healthcare infrastructure and antibiotic selection pressure. At inference time, Kazakhstan's macroeconomic indicators are plugged in and the model predicts resistance probabilities for six priority pathogens across all antibiotics present in Kazakhstan's antibiotic consumption data.

## Methodology

**Data sources**

- ATLAS, SIDERO-WT, KEYSTONE — accessed via formal data request through vivli.org
- World Bank indicators — GDP per capita, health expenditure % GDP, population density
- Kazakhstan antibiotic consumption (DDD per 1000 inhabitants per day, 2017–2023) — extracted from published Kazakhstani literature

**Feature set**

Six features: organism, antibiotic, year, GDP per capita, health expenditure (% GDP), population density. Country name and database source are deliberately excluded.

**Training and validation**

Records collected up to and including 2020 form the training set. Records from 2021–2022 form the validation set. An explicit leakage audit confirmed zero isolate ID overlap across all temporal splits.

Three models were compared: Random Forest as a baseline, XGBoost, and LightGBM. Class imbalance (80.4% susceptible / 19.6% resistant) was handled via `scale_pos_weight = 4.09`.

Zero-shot generalizability was assessed using Leave-One-Country-Out (LOCO) validation across six proxy countries chosen for their epidemiological and geographic similarity to Kazakhstan: Russia, Turkey, Mexico, Belarus, Ukraine, and Bulgaria. Russia served as the primary proxy.

A 5-fold rolling temporal cross-validation was run with genuine year-window boundaries — each fold trains on progressively more historical data and validates on the subsequent period — to assess result stability over time.

## Results

| Model | Val AUC | Val AUPRC |
|---|---|---|
| Random Forest | 0.8518 | 0.6107 |
| XGBoost | 0.8350 | 0.5208 |
| **LightGBM** | **0.8673** | **0.5816** |

LightGBM converged at round 1304 with early stopping. The resistant class recall was 0.82 — prioritizing detection of truly resistant isolates over precision, which is the clinically relevant trade-off.

**LOCO validation** across proxy countries: mean AUC 0.8695, with Russia (the primary Kazakhstan proxy) achieving AUC 0.8793.

**5-fold rolling CV:** mean AUC 0.8744 ± 0.0126. Performance declined from Fold 1 (AUC 0.895, training on 2004–2006) to Fold 5 (AUC 0.861, training on 2004–2018), suggesting the model captures temporal drift in resistance patterns.

**Top Kazakhstan resistance predictions (matched antibiotics only):**

| Organism | Antibiotic | Predicted Resistance Probability |
|---|---|---|
| *Klebsiella pneumoniae* | Ampicillin | 0.997 |
| *Enterococcus faecium* | Ampicillin | 0.967 |
| *Pseudomonas aeruginosa* | Ampicillin | 0.965 |
| *Enterococcus faecium* | Gentamicin | 0.962 |
| *Enterococcus faecium* | Amoxycillin/clavulanate | 0.951 |

25 of 41 Kazakhstan antibiotic entries (64.1%) were matched to the global surveillance vocabulary. The remaining 14 were either regionally specific drugs absent from global AMR databases or antibiotics with naming conventions that could not be safely harmonized.

Feature importance analysis showed antibiotic identity and organism species dominate prediction (normalized gain 1.000 and 0.707 respectively). Macroeconomic features contribute at the margin — consistent with their role as country-level proxies rather than direct resistance determinants.

## Limitations

- Kazakhstan macro data only covers 2017–2023; the global training dataset spans 2004–2023
- MIC breakpoints use simplified approximations with a fallback threshold of 8 µg/mL
- Intermediate SIR category was discarded, which may exclude borderline clinical cases
- 14 Kazakhstan antibiotics (35.9%) could not be matched to the global vocabulary
- The LOCO proxy assumption (Russia ≈ Kazakhstan) is imperfect — no proxy shares Kazakhstan's exact epidemiological environment
- No direct validation against Kazakhstan clinical laboratory data was possible

## Data Access

This repository contains no raw data. See `data/README.md` for instructions on obtaining each dataset.

ATLAS, SIDERO-WT, and KEYSTONE require a formal data request through [vivli.org](https://vivli.org). World Bank indicators are freely available at [data.worldbank.org](https://data.worldbank.org).

## License

MIT License — see [LICENSE](LICENSE) for details. The underlying surveillance data is subject to separate data use agreements via vivli.org and is not redistributed here.
