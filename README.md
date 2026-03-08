# FDA FAERS Drug Safety Prediction & DDI Signal Detection

Predictive modeling and pharmacovigilance analysis using the FDA Adverse Event Reporting System (FAERS) Q1–Q4 2025 data.

## Project Overview

This project has two components:

1. **Adverse Event Severity Prediction** — Binary classification models (Logistic Regression, Decision Tree, Random Forest, KNN) predicting whether an adverse drug event will result in a serious outcome (death, hospitalization, life-threatening, disability, congenital anomaly, or required intervention).

2. **Drug-Drug Interaction (DDI) Signal Detection** — Reporting Odds Ratio (ROR) analysis across 1.6M patient reports to identify potential drug-drug interactions, validated against DrugBank's known DDI database.

## Key Results

- **Best classifier**: Random Forest (ROC AUC = 0.850, Balanced Accuracy = 0.774)
- **DDI signals detected**: 19,741 out of 141,097 drug pairs analyzed
- **High-confidence signals**: 1,366 (after quality filtering)
- **Novel DDI candidates**: 108 (both drugs known in DrugBank, but pair not a documented interaction)
- **DrugBank validation rate**: 14.7% of priority signals matched known DDIs

## Data Sources

| Source | Access | Notes |
|--------|--------|-------|
| [FDA FAERS](https://fis.fda.gov/extensions/FPD-QDE-FAERS/FPD-QDE-FAERS.html) | Public | Download Q1–Q4 2025 ASCII files |
| [NIH RxNorm API](https://rxnav.nlm.nih.gov/REST) | Public | Drug name standardization |
| [DrugBank](https://go.drugbank.com/) | Licensed | DDI validation reference (not redistributed here) |

## Reproducing the Analysis

### Prerequisites

```
pip install pandas numpy scikit-learn matplotlib seaborn scipy
```

### Data Setup

1. Download FAERS quarterly ASCII files from the FDA link above
2. Extract to a local directory and update `dataPath` / `base_path` in the notebook
3. (Optional) Download DrugBank XML for DDI validation — requires a free academic license

### Running

Open and run the Jupyter notebook sequentially. The RxNorm API mapping step takes 2–4 hours on first run; results are cached to `rxnorm_mapping_cache.json` for subsequent runs.

## Generated Files

Files tracked via Git LFS (large):

| File | Description | Rows |
|------|-------------|------|
| `FAERS25Q2_CLEANED.csv` | Cleaned patient-level dataset | 223K |
| `FAERS_DRUG_PAIRS_RXNORM.csv` | RxNorm-standardized drug pairs | Large |
| `FAERS_DDI_ROR_ALL.csv` | ROR statistics for all pairs | 141K |
| `FAERS_DDI_SIGNALS.csv` | Flagged DDI signals | 19.7K |
| `rxnorm_mapping_cache.json` | Cached RxNorm API results | — |

Files tracked normally (small):

| File | Description | Rows |
|------|-------------|------|
| `FAERS_DDI_SIGNALS_HIGH_CONFIDENCE.csv` | Vetted signals (no quality flags, N≥100) | 1,366 |
| `FAERS_DDI_PRIORITY_LIST.csv` | Top signals for validation | 300 |
| `FAERS_DDI_VALIDATED.csv` | Signals with DrugBank validation status | 300 |
| `FAERS_DDI_KNOWN.csv` | Confirmed known DDIs | 44 |
| `FAERS_DDI_NOVEL.csv` | Novel DDI candidates | 108 |

## Methodology

### Severity Prediction
- Target: Serious outcome (binary) per FDA definitions
- Features: Age, sex, drug route, number of reactions/indications/therapies, dose information
- Class balancing via `class_weight='balanced'`

### DDI Signal Detection
- Metric: Reporting Odds Ratio (ROR) with 95% confidence intervals
- Signal criteria: ROR > 2.0, CI lower bound > 1.0, Benjamini-Hochberg FDR-adjusted p < 0.05
- Quality filters: Serious rate outliers, CI width, sample size, veterinary drug exclusion

## Known Limitations

- FAERS is a voluntary reporting system — subject to reporting bias
- Disproportionality (ROR) does not establish causation
- Confounding by indication (e.g., immunosuppressant patients are inherently sicker)
- International drug name variants may cause imperfect RxNorm matching (91.9% mapped)
- DrugBank-derived data is not included due to licensing restrictions