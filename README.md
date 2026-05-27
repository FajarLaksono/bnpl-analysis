# Predictive Modeling of Usage Frequency in Buy Now, Pay Later (BNPL) Services

Hurdle model framework analyzing BNPL adoption, usage intensity, and missed payment risk using the CFPB Making Ends Meet survey (Samples 3-6, Jan 2022 - Jan 2025, ~10K respondents).

## Key Findings

- **Adoption plateaued** at ~25-34% (survey-weighted) across harmonized waves (2024-2025). An apparent growth spurt from 2023 is largely an artifact of a survey wording change.
- **Usage intensity stable**: ~55% light (1-2x/yr), ~22% medium (3-6x), ~23% heavy (>6x).
- **Missed payments affect 8-9%** of BNPL users, concentrated in a heavy/stressed cluster (37.5% of users, 20.6% missed payment rate).
- **Missed payment prediction** achieves ROC-AUC 0.837 (Logistic) / 0.862 (XGBoost). Logistic regression with class weighting is the only actionable model (F1=0.322).
- **Conditional frequency prediction** fails to beat naive baseline - usage intensity drivers are not captured in available survey variables.

## Methodology

9 models across 3 tasks + 1 unsupervised segmentation:

| Task | Models | Dataset | Type |
|------|--------|---------|------|
| Stage 1: Adoption | Logistic + XGBoost + LightGBM | `df_4wave_harmonized` (~11K) | Binary |
| Stage 2: Intensity | Multinomial Logit + XGBoost | BNPL users only (~3K) | Multiclass |
| Missed Payment | Logistic + XGBoost + LightGBM | `df_2wave` (~5.7K) | Binary, rare event |
| Segmentation | K-Means (GMM optional) | BNPL users only | Unsupervised |

Survey weights used in all descriptive stats and linear models. Cluster-robust SEs by `id` for pooled panel data. SHAP for tree model interpretation.

## Repository Structure

```
├── notebooks/
│   └── bnpl-usage-frequency-analysis.ipynb   # Main analysis notebook
├── docs/
│   ├── techspec/
│   │   ├── bnpl-datasets.md                   # Dataset mapping & question evolution
│   │   ├── bnpl-feature-dictionary.md         # All 1,073 variables across 23 categories
│   │   └── bnpl-usage-frequency-analysis.md   # Technical spec
│   ├── white-paper.md                         # Full technical white paper
├── data/raw/                                  # Raw CSV data from CFPB ZIPs
├── requirements.txt
└── .env                                       # Dataset download URLs (not tracked)
```

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Create `.env` with CFPB dataset download URLs (see `dataset_links.txt` for reference).

## Data

Source: [CFPB Making Ends Meet Survey - Public Data](https://www.consumerfinance.gov/data-research/making-ends-meet-survey-data/public-data/)

| Sample | Waves | Respondents | CSV |
|--------|-------|-------------|-----|
| S3 | S3W1 + S3W2 (Jan 2022-2023) | 2,125 | `MEM_S3W1W2_PUF.csv` |
| S4 | S4W1 + S4W2 (Jan 2023-2024) | 2,136 | `MEM_S4W1W2_PUF.csv` |
| S5 | S5W1 + S5W2 (Jan 2024-2025) | 3,113 | `MEM_S5W1W2_PUF.csv` |
| S6 | S6W1 (Jan 2025) | 2,630 | `MEM_S6W_PUF.csv` |

## Quality Gates

- Feature selection on training set only (no leakage)
- `random_state=42` for all splits and models
- Survey weights used in all descriptive stats and linear models
- Cluster-robust SEs by `id` for pooled panel models
- SHAP for tree model feature importance
- Frequency charts: `fillna(0)` before grouping, `.reindex(columns=[0,1,2,3])` for missing categories

## References

- CFPB Making Ends Meet Survey: methodology and public use files
- Chen & Guestrin (2016). XGBoost: A Scalable Tree Boosting System
- Ke et al. (2017). LightGBM: A Highly Efficient Gradient Boosting Decision Tree
- Lundberg & Lee (2017). A Unified Approach to Interpreting Model Predictions (SHAP)
