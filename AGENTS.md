# BNPL Analysis — Project Context

## Overview

This project analyzes Buy Now Pay Later (BNPL) usage patterns using the CFPB Making Ends Meet survey (Samples 3–6, Jan 2022 – Jan 2025).

**Key documents:**
- `docs/techspec/bnpl-datasets.md` — dataset mapping, question evolution, merge strategy
- `docs/techspec/bnpl-feature-dictionary.md` — all 1,073 variables across 23 categories
- `docs/techspec/bnpl-usage-frequency-analysis.md` — technical spec for the notebook
- `notebooks/bnpl-usage-frequency-analysis.ipynb` — the main analysis notebook

---

## Data Sources

| Sample | Waves | Rows | Columns | CSV File (inside ZIP) |
|--------|-------|------|---------|-----------------------|
| S3 | S3W1 (Jan 2022) + S3W2 (Jan 2023) | 2,125 | 404 | `MEM_S3W1W2_PUF.csv` |
| S4 | S4W1 (Jan 2023) + S4W2 (Jan 2024) | 2,136 | 396 | `MEM_S4W1W2_PUF.csv` |
| S5 | S5W1 (Jan 2024) + S5W2 (Jan 2025) | 3,113 | 491 | `MEM_S5W1W2_PUF.csv` |
| S6 | S6W1 (Jan 2025) only | 2,630 | 346 | `MEM_S6W_PUF.csv` |

**File organization:** ZIPs at `docs/cfpb_making-ends-meet_data-sample-{3,4,5,6}.zip`, URLs read from `.env`.
**Format:** Wide-format CSV (one row per respondent, `w2` prefix for Wave 2 columns).

---

## BNPL Question Evolution

| Wave | Date | Frequency Question (CSV col) | Scale | Wording |
|------|------|-----------------------------|-------|---------|
| S3W1 | Jan 2022 | Q85 (`q85`) | Binary yes/no | First BNPL introduction |
| S3W2 | Jan 2023 | Q28 (`w2q28`) | 1=Never, 2=1-2, 3=3-6, 4=>6 | "four installments" |
| S4W1 | Jan 2023 | Q71 (`q71`) | Same 4-point | Same wording |
| S4W2 | Jan 2024 | Q25 (`w2q25`) | Same 4-point | **"four or fewer"** installments |
| S5W1 | Jan 2024 | Q69 (`q69`) | Same 4-point | Same as S4W2 |
| S5W2 | Jan 2025 | Q37 (`w2q55`) + Q48 (`w2q30h`) | Same 4-point + missed payment checkbox | Same + missed payment |
| S6W1 | Jan 2025 | Q83 (`q77`) + Q51 (`q45h`) | Same 4-point + missed payment checkbox | Same structure |

**Key fact:** Wording changed at S4W2 from "four installments" to "four or fewer installments". S4W2+S5W1+S5W2+S6W1 have **identical wording** — used for pooled predictive modeling.

---

## Merged DataFrames

| DataFrame | Waves | Rows | Purpose |
|-----------|-------|------|---------|
| `df_7wave_adopt` | S3W1+S3W2+S4W2+S5W1+S5W2+S6W1 | ~15K | Adoption trend (6 waves, 3 years; name retains `7wave` for notebook legacy) |
| `df_5wave_freq` | S3W2+S4W2+S5W1+S5W2+S6W1 | ~13K | Frequency trend (older wording annotated) |
| `df_4wave_harmonized` | S4W2+S5W1+S5W2+S6W1 | ~11K | Predictive modeling (identical wording) |
| `df_2wave` | S5W2+S6W1 | ~5.7K | Missed payment analysis |

---

## Canonical Column Mapping

| Canonical | S3W1 | S3W2 | S4W1 | S4W2 | S5W1 | S5W2 | S6W1 |
|-----------|------|------|------|------|------|------|------|
| `bnpl_adopt` | `q85`>0 | `w2q28`>0 | `q71`>0 | `w2q25`>0 | `q69`>0 | `w2q55`>0 | `q77`>0 |
| `bnpl_freq` | — | `w2q28` | `q71` | `w2q25` | `q69` | `w2q55` | `q77` |
| `missed_bnpl` | — | — | — | — | — | `w2q30h` | `q45h` |
| `id` | `id` | `id` | `id` | `id` | `id` | `id` | `id` |
| `age_bin` | `age_bin` | `age_bin` | `age_bin` | `age_bin` | `age_bin` | `age_bin` | `age_bin` |
| `sex` | `sex` | `sex` | `sex` | `sex` | `sex` | `sex` | `sex` |
| `race` | `race` | `race` | `race` | `race` | `race` | `race` | `race` |
| `education` | `education` | `education` | `education` | `education` | `education` | `education` | `education` |
| `partner` | `partner` | `partner` | `partner` | `partner` | `partner` | `partner` | `partner` |
| `income` | derived | derived | derived | derived | derived | derived | `q10` |
| `fwb` | `fwb` | `fwb` | `fwb` | `fwb` | `fwb` | `fwb` | `fwb` |
| `housing` | `housing` | `housing` | `housing` | `housing` | `housing` | `housing` | `housing` |
| `military` | `military` | `military` | `military` | `military` | `military` | `military` | `military` |
| `q106` | `q106` | `q106` | `q106` | `q106` | `q106` | `q106` | `q106` |
| `q107` | `q107` | `q107` | `q107` | `q107` | `q107` | `q107` | `q107` |
| `financial_stress` | — | — | `q14` | `q12` | `q12` | `q12` | `q45` |

---

## Feature Dictionary Summary

Key variable groups (see `docs/techspec/bnpl-feature-dictionary.md` for all 1,073 variables):

| Group | Key Variables | Notes |
|-------|---------------|-------|
| Survey Administration | `ID`, `weight` variants | Standalone & combined weights; paper/online variants |
| Financial Wellbeing | `fwb` (composite score, 14–82), `q1a`–`q1e`, `q2a`–`q2b` | **Used as SES proxy** |
| Employment | `q3a1`–`q3a7`, `q3b1`–`q3b7` | Check-all-that-apply work status |
| Demographics | `age_bin`, `sex`, `race`, `education`, `partner`, `housing`, `military` | Available across all samples |
| Income | `income` (binned 1–5) | **76–83% missing — DO NOT USE in pooled models** |
| BNPL | `bnpl_freq`, `missed_bnpl` | Primary targets |
| Financial Difficulty | `financial_stress`, `q42`–`q46` | Bill/expense difficulty |
| Credit Cards | `q95`–`q88` | CC ownership, balances, late fees |
| Medical Debt | `q50`–`q58` | Past-due bills, collections |
| Unexpected Expenses | `q33/x_a1`–`x_i2`, `q37/x_a1`–`x_i2`, `w2q20/x_a1`–`x_i2` | Indicator + amount pairs |
| Income Shocks | `q34/x_a1`–`x_l2`, `q38/x_a1`–`x_l2`, `w2q21/x_a1`–`x_k2` | Indicator + amount pairs |

---

## Key Data Cleaning Decisions

| Decision | Rationale |
|----------|-----------|
| `income` dropped from pooled features | 76–83% missing; `fwb` used as SES proxy instead |
| `bnpl_freq` → `fillna(0)` | 24% missing = never-users (not asked / skip logic) |
| `bnpl_freq` recoded to 0-indexed | 0=Never, 1=1-2, 2=3-6, 3=>6 |
| Demographics mode-imputed | All <7% missing |
| `missed_bnpl` → `dropna()` | Skip-logic for non-users; only asked of BNPL users |
| Question mode splits | S5W2/S6W1 have paper+online and online-only weight variants — select the variant matching the question mode |

**Weight regimes:**
- **Standalone** (per-wave descriptive): S3W1=`weight`, S3W2=`w2weight`, S4W1=`weight`, S4W2=`w2weight`, S5W1=`weight`, S5W2=`w2weight_paper`/`w2weight_online`, S6W1=`weight_paper`/`weight_online`
- **Combined** (pooled modeling): S4W2=`w2comb_weight`, S5W1=`comb_weight`, S5W2=`w2comb_weight_paper`/`w2comb_weight_online`, S6W1=`comb_weight_paper`/`comb_weight_online`

---

## Analysis Methodology

### Predictive Models (9 total: 8 supervised + 1 unsupervised)

| Task | Models | Dataset | Class Type |
|------|--------|---------|------------|
| Stage 1: Adoption | Logistic + XGBoost + LightGBM | `df_4wave_harmonized` | Binary (Ever vs Never) |
| Stage 2: Intensity | Multinomial Logit + XGBoost | BNPL users only | Multiclass (1-2 / 3-6 / >6) |
| Missed Payment | Logistic + XGBoost + LightGBM | `df_2wave` | Binary (rare event ~10-15%) |
| Segmentation | K-Means (GMM optional) | BNPL users only | Unsupervised (k=2..5) |

**No deep learning models** — data too small (~8K rows, tabular).

### Hurdle Model
Stage 1 predicts adoption (binary), Stage 2 predicts conditional frequency among users. Separates "ever trying" from "using heavily."

### Evaluation Metrics
- Stage 1: ROC-AUC, PR-AUC, F1
- Stage 2: Accuracy, Macro F1, Ordinal MAE
- Missed Payment: ROC-AUC, PR-AUC, Precision-Recall curve, Calibration

### Interpretation
- Linear models: odds ratios with 95% CI + VIF check for multicollinearity
- Tree models: SHAP beeswarm summary plots
- Panel correction: cluster-robust SEs by `id` for pooled 4-wave models

### Feature Selection (train-set only — no leakage)
- Continuous features: Weighted Kruskal-Wallis H-test
- Categorical features: Rao-Scott corrected Chi-square
- Effect size: Cramér's V (categorical), Epsilon-squared (continuous)

---

## Target Variable Coding

After cleaning in the notebook:

| Variable | Coding |
|----------|--------|
| `bnpl_freq` | 0=Never, 1=1-2 times, 2=3-6 times, 3=>6 times |
| `bnpl_adopt` | 0=Never, 1=Ever used (i.e., `bnpl_freq > 0`) |
| `bnpl_freq_cond` | 1=1-2, 2=3-6, 3=>6 (users only, NaN for non-users) |
| `missed_bnpl` | 0=No missed payment, 1=Missed BNPL payment |

---

## Quality Gates

- Feature selection performed on training set only
- `random_state=42` for all splits and models
- Survey weights used in all descriptive stats and linear models
- Cluster-robust SEs by `id` for pooled 4-wave panel data
- SHAP for tree model feature importance
- Frequency charts: `fillna(0)` before grouping, `.reindex(columns=[0,1,2,3])` to guard against missing categories

---

## Skill Notes

- `feature-engineering` skill — currently references Azure VM trace data (`vm_core_count_bucket`, `waste_fraction`, etc.). **Not relevant to BNPL analysis.**
- `model-evaluation` skill — also for Azure VM project (QA thresholds for CPU regression etc.). **Not relevant to BNPL analysis.**
- `root-cause-analysis` skill — general-purpose debugging methodology; fine to use.
- If you need BNPL-specific skills, create them under `.opencode/skills/`.
