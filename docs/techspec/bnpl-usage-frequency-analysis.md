# BNPL Usage Frequency Analysis — Technical Specification

## Overview

This document defines the technical specification for `notebooks/bnpl-usage-frequency-analysis.ipynb`, which analyzes Buy Now Pay Later (BNPL) usage frequency trends across the CFPB Making Ends Meet survey waves and builds predictive models to understand BNPL adoption drivers and missed-payment risk.

## Scope

### Analysis Scope
| Dimension | Coverage |
|-----------|----------|
| **BNPL frequency trend** | S4W2 (2024) → S5W1 (2024) → S5W2 (2025) → S6W1 (2025) — 4 waves, identical wording |
| **Missed payment analysis** | S5W2 + S6W1 — 2 waves with BNPL missed-payment checkbox |
| **Feature selection** | Train set only — weighted statistical screening of covariates (no data leakage) |
| **Predictive modeling (frequency)** | Hurdle model — Stage 1: BNPL adoption (binary), Stage 2: conditional frequency (multiclass) |
| **Predictive modeling (missed payment)** | Logistic Regression + XGBoost + LightGBM (binary, imbalanced) |
| **Segmentation** | K-Means (GMM as optional diagnostic) on BNPL users for latent profile discovery |
| **DL models** | None — data size (~8K rows) is too small for deep learning on tabular data |

### Data Sources
- CSVs extracted from `docs/cfpb_making-ends-meet_data-sample-4.zip` → `MEM_S4W1W2_PUF.csv`
- CSVs extracted from `docs/cfpb_making-ends-meet_data-sample-5.zip` → `MEM_S5W1W2_PUF.csv`
- CSVs extracted from `docs/cfpb_making-ends-meet_data-sample-6.zip` → `MEM_S6W_PUF.csv`
- See `docs/techspec/bnpl-datasets.md` for full dataset mapping

## Model Suitability Rationale

Given the data constraints (~8K rows, ~15–20 mostly categorical features, class imbalance), DL models (MLP, TabNet, Transformers) are not suitable — tree-based models consistently outperform neural networks on tabular data under 100K rows.

### Final Model Selection (9 models)

Each task picks a **linear model** (inference: survey-weighted odds ratios) + **1–2 tree models** (prediction: SHAP interpretability). Multiple tree variants per task are avoided unless they serve distinct purposes (e.g., cross-validating feature rankings for binary tasks with sufficient N).

| Task | Selected Models | Total | Reasoning |
|------|----------------|-------|-----------|
| Stage 1: Adoption | Logistic + XGBoost + LightGBM | 3 | ~8K rows supports two tree variants for cross-validation of feature importance |
| Stage 2: Intensity | Multinomial Logit + XGBoost | 2 | ~3K users only; one tree variant is sufficient; LightGBM risks overfitting on this N |
| Missed Payment | Logistic + XGBoost + LightGBM | 3 | ~5.7K rows, rare event (~10-15%); two tree variants reinforce confidence in feature rankings |
| Segmentation | K-Means | 1 | Simple, interpretable clusters; GMM as optional diagnostic if silhouette < 0.3 |
| **Total** | | **9** | |

### Models Excluded & Why

| Model | Reason Excluded |
|-------|-----------------|
| Random Forest | Robust but consistently underperforms XGBoost/LightGBM on this data size. Contribution is redundant. |
| CatBoost | Native categorical handling is valuable but XGBoost + one-hot encoding achieves equivalent results. A 4th model per task provides diminishing returns. |
| MLP / TabNet / Transformer | Data too small (~8K rows). TabNet/Transformers need 100K+. MLP is borderline but unlikely to beat gradient boosting. |
| GMM | Useful but K-Means is sufficient for initial discovery. GMM adds complexity without guaranteed insight. |
| DBSCAN | Features are low-dimensional and globular; density-based clustering unlikely to find meaningful structure. |

## Notebook Layout

The notebook follows the standard project layout established in `notebooks/heart_disease.ipynb`.

### Section 0 — Table of Contents
Hyperlinked table of contents mapping all sections 1–9.

### Section 1 — Summary
Key highlights covering dataset overview, analysis approach, key findings, practical implications, and limitations. Written after all analysis is complete.

### Section 2 — Preparation

#### 2.1. Import Libraries
Import and print versions of:
- `pandas`, `numpy` — data processing
- `matplotlib`, `seaborn` — visualization
- `scipy.stats` — statistical tests (weighted variants)
- `sklearn` — ML pipeline (train_test_split, GridSearchCV, StandardScaler, metrics, cluster)
- `xgboost` — primary gradient boosting
- `lightgbm` — secondary gradient boosting (cross-validate feature rankings)
- `statsmodels` — logistic regression with survey weights
- `shap` — feature importance
- `imblearn` — class imbalance handling (SMOTE)
- `ipynbname` + `pathlib.Path` — notebook path detection

#### 2.2. Load Dataset
- Extract CSVs from ZIP files in `docs/`
- Load into DataFrames: `df_s4`, `df_s5`, `df_s6`

#### 2.3. Data Mapping & Merging
Map survey columns to canonical variable names:

| Canonical Name | S4W2 | S5W1 | S5W2 | S6W1 |
|----------------|------|------|------|------|
| `bnpl_freq` | `w2q25` | `q69` | `w2q37` | `q83` |
| `missed_bnpl` | — | — | `w2q48` | `q51` |
| `id` | `id` | `id` | `id` | `id` |
| `age_bin` | `age_bin` | `age_bin` | `age_bin` | `age_bin` |
| `sex` | `sex` | `sex` | `sex` | `sex` |
| `race` | `race` | `race` | `race` | `race` |
| `education` | `education` | `education` | `education` | `education` |
| `partner` | `partner` | `partner` | `partner` | `partner` |
| `income` | derived | derived | derived | compute from `q10` |
| `fwb` | `fwb` | `fwb` | `fwb` | `fwb` |
| `housing` | `housing` | `housing` | `housing` | `housing` |
| `military` | `military` | `military` | `military` | `military` |
| `q106` | `q106` | `q106` | `q106` | `q106` |
| `q107` | `q107` | `q107` | `q107` | `q107` |
| `q12`/`q45` | `q12` | `q12` | `q12` | `q45` |

Merge into two DataFrames:
- **`df_4wave`** — rows: S4W2, S5W1, S5W2, S6W1 (BNPL frequency only, all covariates)
- **`df_2wave`** — rows: S5W2, S6W1 (BNPL frequency + missed payment)

### Section 3 — Exploratory Data Analysis

Standard EDA per wave:
- **3.1** Dataset preview (`head()`)
- **3.2** Shape (rows, columns)
- **3.3** Info (dtypes, memory)
- **3.4** Missing values (`isnull().sum()`)
- **3.5** Duplicate check
- **3.6** Summary — column explanations, BNPL question evolution reference
- **3.7** Class balance check — BNPL frequency distribution + missed payment prevalence per wave

### Section 4 — Data Cleaning

Cleaning decisions:
- Recode BNPL frequency to ordinal: Never=0, 1-2=1, 3-6=2, >6=3
- Create hurdle targets:
  - `bnpl_adopt` — binary: 0=Never, 1=Ever used (used for Stage 1)
  - `bnpl_freq_cond` — recoded to 1/2/3 for users only (used for Stage 2)
- Derive `income` for S6W1 from raw `q10` using same band mapping
- Handle missing values in covariates (listwise deletion or mode imputation depending on rate)
- No inter-sample dedup (separate samples = separate respondents)
- Record survey weight columns per sample. Two regimes:
  - **Standalone weights** (for §5 descriptive analysis per wave): S4W2=`w2weight`, S5W1=`weight`, S5W2=`w2weight_paper`/`w2weight_online`, S6W1=`weight_paper`/`weight_online`. Each wave represents its own population.
  - **Combined weights** (for §6 pooled modeling): S4W2=`w2comb_weight`, S5W1=`comb_weight`, S5W2=`w2comb_weight_paper`/`w2comb_weight_online`, S6W1=`comb_weight_paper`/`comb_weight_online`. The merged dataset is representative as a whole.
- For paper/online splits, select the variant matching the question mode (paper+online or online-only).

### Section 5 — Descriptive Analysis

#### 5.1. BNPL Frequency Distribution
- **5.1.1** Weighted frequencies using standalone weights per wave (see §4 for mapping)
- **5.1.2** Unweighted frequencies for comparison (raw survey counts)

#### 5.2. Trend Analysis
- **5.2.1** Adoption over time: % "Ever used BNPL" (frequency > 0) by wave, with 95% CIs (standalone weights)
- **5.2.2** Category shifts: stacked bar of frequency categories per wave (standalone weights)
- **5.2.3** Missed payment rate among BNPL users (S5W2 + S6W1) (standalone weights)

#### 5.3. Survey-Weighted Estimates
- Weighted vs unweighted comparison table using standalone weights
- Demonstrate impact of weighting on adoption rate estimates

#### 5.4. Feature Selection & Relevance Analysis

**Important:** To prevent data leakage, perform train-test split (80/20, stratified by wave) *before* statistical screening. Run all screening on training set only.

##### 5.4.1. Train-Test Split
- Split `df_4wave` into `train_4wave` and `test_4wave` (80/20, stratified by wave)
- Split `df_2wave` into `train_2wave` and `test_2wave` (80/20, stratified by wave)
- Hold test sets aside for Section 6 evaluation

##### 5.4.2. Weighted Statistical Screening
All tests must use survey weights to account for complex sampling design.

| Feature Type | Test | Null Hypothesis | Weighted Variant |
|-------------|------|-----------------|-----------------|
| Continuous (e.g., `fwb`, `age_bin`, `income`) | Weighted Kruskal-Wallis H-test | No difference in median across BNPL frequency groups | Apply `weight` as frequency weights |
| Categorical (e.g., `race`, `education`, `sex`, `housing`) | Weighted Chi-square test | Independence from BNPL frequency | Rao-Scott second-order correction |

##### 5.4.3. Effect Size Ranking
- Categorical: Cramér's V (weighted)
- Continuous: Epsilon-squared (from weighted Kruskal-Wallis)
- Summary table: Feature \| Test Stat \| p-value \| Effect Size \| Rank

##### 5.4.4. Visualization
- Grouped bar charts: BNPL frequency × top 4 categorical features (weighted)
- Violin/box plots: BNPL frequency × top 2 continuous features (weighted)

##### 5.4.5. Interpretation
- Which features best differentiate Never-users vs Heavy-users?
- Shortlist top features for modeling (Section 6)

### Section 6 — Predictive Modeling

#### 6.1. Modeling BNPL Frequency (Hurdle Model)

This is a two-stage approach: first predict whether someone adopts BNPL at all, then predict usage intensity among adopters. This separates the conceptually different processes of "ever trying" vs "using heavily."

##### 6.1.1. Data Preparation
- **Dataset:** `train_4wave` (training), `test_4wave` (testing)
- **Stage 1 target:** `bnpl_adopt` (binary: 0=Never, 1=Ever)
- **Stage 2 target:** `bnpl_freq_cond` (ordinal: 1=1-2 times, 2=3-6 times, 3=>6 times) — subset of users only
- **Features:** top covariates from §5.4 + interaction terms
- Interaction terms:
  - `age_bin × income`
  - `fwb × housing`
  - `sex × partner`
  - `income × education`
- **Multicollinearity check:** After fitting each linear model, check VIF for all terms. Drop any interaction with VIF > 10 (high multicollinearity with main effects can destabilize coefficient estimates).
- Feature scaling: StandardScaler (for linear models)
- **Survey weights:** Use combined weights for pooled modeling (see §4 for mapping). The merged dataset is representative as a whole.
- **Panel correction:** S5 appears twice (W1 + W2). Use **cluster-robust standard errors by `id`** for all linear models.

##### 6.1.2. Stage 1: BNPL Adoption (Binary Classification)

###### Model Selection Rationale

| Model | Rank | Reasoning |
|-------|------|-----------|
| **XGBoost** | 1 — Primary | Best accuracy for small tabular data. Handles missing values and imbalance natively. SHAP for interpretability. |
| **Logistic Regression** | 2 — Inference | Survey weights produce unbiased odds ratios, answering "what drives BNPL adoption?" |
| **LightGBM** | 3 — Cross-validation | Second tree variant with ~8K rows is feasible; cross-validates feature rankings against XGBoost. |
| Random Forest | 4 — Excluded | Consistently underperforms XGBoost on this data size; adds no unique insight. |
| CatBoost | 5 — Excluded | Redundant to XGBoost + one-hot encoding. 4th model provides diminishing returns. |

###### Models

| Model | GridSearchCV Parameters |
|-------|------------------------|
| **Baseline** | Predict majority class (Never) — floor reference |
| **Logistic Regression** | `C`: [0.01, 0.1, 1, 10], `penalty`: [l1, l2], `class_weight`: [balanced, None] |
| **XGBoost** | `n_estimators`: [100, 200], `max_depth`: [3, 6, 9], `learning_rate`: [0.01, 0.1], `subsample`: [0.8, 1.0], `colsample_bytree`: [0.8, 1.0], `scale_pos_weight`: [1, ratio] |
| **LightGBM** | `n_estimators`: [100, 200], `max_depth`: [3, 6, 9], `learning_rate`: [0.01, 0.1], `subsample`: [0.8, 1.0], `class_weight`: [balanced, None] |

- Logistic Regression: use survey weights in loss function
- All tree models: early stopping on validation set

###### Evaluation

| Metric | Rationale |
|--------|-----------|
| ROC-AUC | Overall separability |
| PR-AUC | Better for imbalanced adoption rate? |
| F1 | Balance precision/recall |
| Confusion Matrix | Threshold-dependent performance |

###### Feature Importance
- Logistic Regression: coefficient plot with 95% CI (odds ratios)
- XGBoost / LightGBM: SHAP beeswarm summary plot
- Comparison table of top 10 features across all models

##### 6.1.3. Stage 2: Conditional BNPL Frequency (Multiclass Classification)

Subset to BNPL users only.

###### Model Selection Rationale

| Model | Rank | Reasoning |
|-------|------|-----------|
| **XGBoost** | 1 — Primary | ~3K rows across 3 classes is near the minimum for tree-based multiclass. XGBoost with strong regularization (max_depth ≤ 6) handles this well. |
| **Multinomial Logistic** | 2 — Inference | Small N + 3 classes means some classes may have <500 rows. Regularized logistic is stable here. Survey weights for unbiased odds ratios. |
| LightGBM | 3 — Excluded | ~3K rows split across 3 classes risks overfitting even with regularization. Insufficient N for a second tree variant. |
| Random Forest / CatBoost | 4 — Excluded | Redundant; adds no value beyond XGBoost at this sample size. |

###### Models

| Model | GridSearchCV Parameters |
|-------|------------------------|
| **Baseline** | Predict majority class among users — floor reference |
| **Multinomial Logistic Regression** | `C`: [0.01, 0.1, 1, 10], `penalty`: [l1, l2] |
| **XGBoost** | `objective`: `multi:softprob`, `n_estimators`: [100, 200], `max_depth`: [3, 6], `learning_rate`: [0.01, 0.1], `subsample`: [0.8, 1.0] |

- Use survey weights in multinomial logistic regression
- **Panel correction:** cluster-robust SEs by `id` for the linear model
- Early stopping on validation set for XGBoost

###### Evaluation

| Metric | Rationale |
|--------|-----------|
| Accuracy | Overall correctness |
| Macro F1 | Equal weight per class (handles imbalance) |
| Ordinal MAE | Penalizes distant misclassifications more |
| Confusion Matrix | Visualize misclassification patterns |
| ROC-AUC (OvR) | Separability per class |

###### Feature Importance
- Multinomial Logit: coefficient plot with 95% CI (odds ratios per class)
- XGBoost: SHAP beeswarm summary plot
- Comparison table of top 10 features across both models

#### 6.2. Modeling Missed BNPL Payment (Binary Classification)

##### 6.2.1. Data Preparation
- **Dataset:** `train_2wave` (training), `test_2wave` (testing)
- **Target:** `missed_bnpl` (0/1)
- **Features:** `bnpl_freq` + top 5 covariates from §5.4 + interaction terms (including `age_bin × bnpl_freq`)
- **Multicollinearity check:** After fitting logistic regression, check VIF for all terms. Drop any interaction with VIF > 10.
- Handle class imbalance: compute class weights or SMOTE
- **Survey weights:** Use combined weights for pooled modeling (S5W2=`w2comb_weight_paper`/`w2comb_weight_online`, S6W1=`comb_weight_paper`/`comb_weight_online`).
- **No panel correction needed** — S5 and S6 are entirely different samples (no repeated respondents)

##### 6.2.2. Model Selection Rationale

| Model | Rank | Reasoning |
|-------|------|-----------|
| **XGBoost** | 1 — Primary | `scale_pos_weight` handles rare event (~10-15%). Best PR-AUC for imbalanced binary classification. |
| **Logistic Regression** | 2 — Inference | Survey weights + odds ratios essential when target is rare — interpretability matters most for missed-payment risk factors. |
| **LightGBM** | 3 — Cross-validation | ~5.7K rows supports a second tree variant. Agreement between XGBoost and LightGBM on feature rankings builds confidence. |
| Random Forest | 4 — Excluded | Robust but XGBoost/LightGBM will outperform. No unique insight. |
| CatBoost | 5 — Excluded | Redundant. |

##### 6.2.3. Models

| Model | GridSearchCV Parameters |
|-------|------------------------|
| **Baseline** | Predict majority class (No missed payment) — floor reference |
| **Logistic Regression** | `C`: [0.01, 0.1, 1, 10], `penalty`: [l1, l2], `class_weight`: [balanced, None] |
| **XGBoost** | Same grid as 6.1.2 + `scale_pos_weight`: [1, ratio_of_neg_to_pos] |
| **LightGBM** | Same grid as 6.1.2, `class_weight`: [balanced, None] |

- Logistic Regression: use survey weights
- All tree models: early stopping on validation set

##### 6.2.4. Evaluation

| Metric | Rationale |
|--------|-----------|
| ROC-AUC | Overall separability |
| PR-AUC | Better for rare event (missed payment) |
| Precision-Recall curve | Threshold selection |
| F1 / F-beta | Balance precision/recall |
| Confusion Matrix | Threshold-dependent performance |
| Calibration curve | Predicted probability reliability |

##### 6.2.5. Feature Importance
- Logistic: odds ratios with CI
- XGBoost / LightGBM: SHAP force plots (individual) + beeswarm (global)
- Key question: Does BNPL frequency remain a significant predictor after controlling for demographics?

#### Model Inventory (Final)

| Task | Baseline | Linear | XGBoost | LightGBM |
|------|----------|--------|---------|----------|
| Stage 1: Adoption | ✓ | Logistic Regression | ✓ | ✓ |
| Stage 2: Intensity | ✓ | Multinomial Logit | ✓ | — |
| Missed Payment | ✓ | Logistic Regression | ✓ | ✓ |

**Total: 8 supervised models** (baselines are floor references, not trained). **+1 unsupervised (K-Means) = 9 total. No DL models.**

### Section 7 — User Segmentation (Unsupervised)

#### 7.1. Data Preparation
- Subset: BNPL users only (bnpl_freq > 0)
- Features: `bnpl_freq_cond`, `fwb`, `income`, `age_bin`, `missed_bnpl`
- Scale with StandardScaler

#### 7.2. K-Means Clustering
- Grid search k=2..5
- Selection criteria: silhouette score + elbow method (inertia)
- Interpret each cluster profile (mean bnpl_freq, % missed payment, mean income, mean fwb, age distribution)
- Label clusters interpretively (e.g., "Strapped heavy users", "Casual healthy users")
- **Optional:** If silhouette < 0.3, run GMM as a diagnostic to check if soft assignment yields better separation.

#### 7.3. Cross-Tabulation
- Cluster × `missed_bnpl`
- Cluster × `bnpl_freq` categories
- Cluster × `income` groups
- Cluster × age distribution

### Section 8 — Key Insights from Modeling

- **8.1** Summary table of final model performance (all models compared side-by-side)
- **8.2** Which BNPL frequency users are most at risk of missed payments?
- **8.3** What non-BNPL factors matter most (age, income, FWB)?
- **8.4** Interaction effects: do certain demographics amplify the frequency→risk relationship?
- **8.5** How do feature rankings differ between Stage 1 (adoption) and Stage 2 (intensity)?
- **8.6** Cluster profiles: what latent segments exist among BNPL users?

### Section 9 — Conclusions and Recommendations

- **9.1** Summary of Findings
- **9.2** Practical Implications (policy, lender risk assessment)
- **9.3** Limitations (survey self-report, 4-point scale granularity, covariate differences across waves, small N for rare events)

## Quality Gates

| Check | Standard |
|-------|----------|
| Execution | Notebook runs end-to-end (papermill-compatible) |
| Data integrity | All CSV columns correctly mapped per bnpl-datasets.md |
| No data leakage | Feature selection (§5.4) performed on training set only |
| ML reproducibility | random_state=42 for all train/test splits and models |
| Feature importance | SHAP used for tree models; Odds Ratios for linear models |
| Survey weights | All descriptive statistics and linear models use appropriate survey weights |
| Panel correction | Cluster-robust standard errors by `id` for all linear models on 4-wave pooled data |
