# Predicting Antimicrobial Resistance Patterns in Kazakhstan Using Zero-Shot Machine Learning

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![Kaggle](https://img.shields.io/badge/Notebook-Kaggle-20BEFF.svg)](https://www.kaggle.com/code/uaisamangeldi/amr-kazakhstan-zeroshot-v2)
[![medRxiv](https://img.shields.io/badge/Preprint-medRxiv-red.svg)](https://www.medrxiv.org)

> A zero-shot machine learning framework for predicting antimicrobial resistance (AMR) patterns in Kazakhstan — a country with no clinical training data in global surveillance databases — using macroeconomic proxy features as surrogates for local epidemiological context.

---

## Overview

Kazakhstan lacks a nationally integrated AMR surveillance infrastructure, making traditional data-driven models inapplicable. This project trains gradient boosting models on three global AMR surveillance databases (ATLAS, SIDERO-WT, KEYSTONE) enriched with World Bank macroeconomic indicators, then applies the trained model to Kazakhstan in a **zero-shot** setting — the model never sees any Kazakhstani clinical data during training.

**Key idea:** By excluding country identity from the feature set and replacing it with macroeconomic proxies (GDP per capita, health expenditure, population density), the model characterizes countries through their healthcare infrastructure rather than their names — enabling generalization to countries absent from training data.

---

## Results Summary

| Model | Val AUC | Val AUPRC |
|---|---|---|
| Random Forest | 0.8518 | 0.6107 |
| XGBoost | 0.8350 | 0.5208 |
| **LightGBM** | **0.8673** | **0.5816** |

**5-Fold Rolling CV:** Mean AUC 0.8744 ± 0.0126 | Mean AUPRC 0.6617 ± 0.0576

**LOCO Validation (Russia as KZ proxy):** AUC 0.8793

**Top Kazakhstan resistance predictions (matched antibiotics):**

| Organism | Antibiotic | Predicted Resistance Probability |
|---|---|---|
| *Klebsiella pneumoniae* | Ampicillin | 0.997 |
| *Enterococcus faecium* | Ampicillin | 0.967 |
| *Pseudomonas aeruginosa* | Ampicillin | 0.965 |
| *Enterococcus faecium* | Gentamicin | 0.962 |
| *Enterococcus faecium* | Amoxycillin/clavulanate | 0.951 |

---

## Repository Structure

```
amr-kazakhstan-zeroshot/
│
├── amr-kazakhstan-zeroshot.ipynb   Main analysis notebook (Kaggle)
├── README.md                       This file
├── requirements.txt                Python dependencies
├── LICENSE                         MIT License
│
└── data/
    └── README.md                   Data sources and access instructions
```

---

## Data Sources

All datasets must be obtained independently. This repository contains no raw data.

| Dataset | Description | Access |
|---|---|---|
| ATLAS | Global AMR surveillance (Pfizer, 2004–2023) | [vivli.org](https://vivli.org) — formal data request required |
| SIDERO-WT | Global AMR surveillance (Shionogi) | [vivli.org](https://vivli.org) — formal data request required |
| KEYSTONE | Global AMR surveillance | [vivli.org](https://vivli.org) — formal data request required |
| World Bank — Global Macro | GDP per capita, health expenditure % GDP, population density | [data.worldbank.org](https://data.worldbank.org) — free download |
| World Bank — Kazakhstan Macro | Kazakhstan-specific indicators | [data.worldbank.org](https://data.worldbank.org) — free download |
| Kazakhstan DID | Antibiotic consumption (DDD/1000/day, 2017–2023) | Extracted from published literature (see paper) |

World Bank indicator codes used:
- `NY.GDP.PCAP.CD` — GDP per capita (current USD)
- `SH.XPD.CHEX.GD.ZS` — Current health expenditure (% of GDP)
- `EN.POP.DNST` — Population density (people per sq. km)

---

## Methods

### Pipeline Overview

```
ATLAS + SIDERO-WT + KEYSTONE
         ↓
   Harmonization & binary encoding (SIR → R=1/S=0/I=discarded)
         ↓
   World Bank macro merge (by country × year)
         ↓
   Temporal split: Train ≤2020 | Val 2021–2022 | Test >2022
         ↓
   Model training (RF / XGBoost / LightGBM)  ← Kazakhstan excluded
         ↓
   LOCO validation on proxy countries
         ↓
   Zero-shot inference on Kazakhstan
```

### Feature Set

| Feature | Type | Rationale |
|---|---|---|
| Organism | Categorical | Species-level resistance biology |
| Antibiotic | Categorical | Drug class and mechanism |
| Year | Numerical | Temporal resistance trends |
| GDP per capita (USD) | Numerical | Healthcare investment capacity |
| Health expenditure (% GDP) | Numerical | Healthcare system strength |
| Population density | Numerical | Transmission and selection pressure |

> **Country name is deliberately excluded** — its presence would prevent zero-shot generalization to unseen countries.

### Zero-Shot Design

The model is trained on 85 countries excluding Kazakhstan. At inference time, Kazakhstan's macroeconomic profile (GDP, health expenditure, population density) and antibiotic consumption data (DDD/1000/day) are used as inputs. The model generalizes based on country characteristics rather than country identity.

LOCO (Leave-One-Country-Out) validation on Russia, Turkey, Mexico, Belarus, Ukraine, and Bulgaria provides indirect evidence of zero-shot generalizability before applying to Kazakhstan.

---

## Reproducibility

### Environment

```bash
pip install -r requirements.txt
```

### Running on Kaggle

The notebook is designed to run on Kaggle with the following datasets attached:

```
/kaggle/input/atlas-csv/atlas.csv
/kaggle/input/ddddddd/sidero_wt.xlsx
/kaggle/input/ddddddd/keystone.xlsx
/kaggle/input/datasets/uaisamangeldi/macrob/       ← World Bank global macro
/kaggle/input/datasets/uaisamangeldi/macro-kz/     ← World Bank Kazakhstan macro
/kaggle/input/datasets/uaisamangeldi/kz-data/      ← Kazakhstan DID
/kaggle/input/datasets/uaisamangeldi/consumption/  ← Global antibiotic consumption
```

Enable GPU accelerator in Kaggle Settings for faster training (LightGBM and XGBoost both support CUDA). The notebook includes CPU fallback for all GPU operations.

---

## Limitations

- Kazakhstan macro data available only 2017–2023; global dataset spans 2004–2023
- MIC breakpoints use simplified CLSI/EUCAST approximations with a fallback threshold of 8 µg/mL
- Intermediate (I) SIR category discarded — may discard clinically borderline isolates
- Of 41 Kazakhstan antibiotic entries, 25 (64.1%) were matched to the global source vocabulary; 14 remained unmatched due to regional drug naming or absence from global surveillance panels
- Feature importance analysis shows antibiotic identity and organism species dominate prediction (normalized gain > 0.7); macroeconomic features contribute at the margin — the model primarily captures globally conserved resistance relationships
- LOCO proxy assumption (Russia ≈ Kazakhstan) is imperfect — no proxy country shares an identical epidemiological environment with Kazakhstan
- No direct validation against Kazakhstan clinical laboratory data was possible

---

## Citation

If you use this code or findings in your work, please cite:

```bibtex
@article{uaisamangeldi2025amr,
  title   = {Predicting Antimicrobial Resistance Patterns in Kazakhstan
             Using Zero-Shot Machine Learning},
  author  = {[Author Names]},
  journal = {medRxiv},
  year    = {2025},
  doi     = {[DOI]}
}
```

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

The underlying surveillance data (ATLAS, SIDERO-WT, KEYSTONE) is subject to separate data use agreements via [vivli.org](https://vivli.org) and is not redistributed here.

---

## Acknowledgements

- ATLAS, SIDERO-WT, and KEYSTONE surveillance data accessed via [vivli.org](https://vivli.org)
- Macroeconomic data from the [World Bank Open Data](https://data.worldbank.org)
- Kazakhstan antibiotic consumption data from published literature (Zhussupova et al. 2021; Semenova et al. 2024; Kassym et al. 2025; Lim et al. 2025)
- Supervised by [Professor Name], Astana IT University
