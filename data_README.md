# Data Sources

This directory contains no raw data. All datasets must be obtained independently.

## Global AMR Surveillance (ATLAS, SIDERO-WT, KEYSTONE)

Access via formal data request at [vivli.org](https://vivli.org).

These databases contain clinical isolate-level antimicrobial susceptibility data:
- **ATLAS** (Pfizer): ~10.7M records, 2004–2023
- **SIDERO-WT** (Shionogi): ~435K records
- **KEYSTONE**: ~1.3M records

Once obtained, place files at the Kaggle input paths specified in the notebook.

## World Bank Macroeconomic Indicators

Download directly from [data.worldbank.org](https://data.worldbank.org).

| Indicator | Code | File |
|---|---|---|
| GDP per capita (current USD) | `NY.GDP.PCAP.CD` | `API_NY.GDP.PCAP.CD_DS2_en_csv_v2.csv` |
| Health expenditure (% of GDP) | `SH.XPD.CHEX.GD.ZS` | `API_SH.XPD.CHEX.GD.ZS_DS2_en_csv_v2.csv` |
| Population density | `EN.POP.DNST` | `API_EN.POP.DNST_DS2_en_csv_v2.csv` |
| Population total | `SP.POP.TOTL` | `API_SP.POP.TOTL_DS2_en_csv_v2.csv` |

## Global Antibiotic Consumption

Download from [Our World in Data](https://ourworldindata.org/antibiotic-resistance):
- Defined daily doses (DDD) of antibiotics per 1,000 inhabitants per day

## Kazakhstan DID Data

Antibiotic consumption data for Kazakhstan (2017–2023) was manually extracted
from the following published sources:

- Zhussupova G. et al. (2021)
- Semenova Y. et al. (2024)
- Kassym L. et al. (2025)
- Lim L. et al. (2025)

The compiled CSV (`kazakhstan_antibiotics_did_2017_2023.csv`) contains columns:
`year`, `antibiotic`, `atc_code`, `aware_category`, `did`
