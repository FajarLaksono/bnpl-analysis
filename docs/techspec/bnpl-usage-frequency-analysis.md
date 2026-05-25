# BNPL Usage Frequency Analysis — Technical Specification

## Overview

This document defines the technical specification for `notebooks/bnpl-usage-frequency-analysis.ipynb`, which analyzes Buy Now Pay Later (BNPL) usage frequency trends across the CFPB Making Ends Meet survey waves and builds predictive models to understand BNPL adoption drivers and missed-payment risk.

## Scope

### Analysis Scope
| Dimension | Coverage |
|-----------|----------|
| **BNPL adoption trend** | S3W1 (2022) → S3W2 (2023) → S4W1 (2023) → S4W2 (2024) → S5W1 (2024) → S5W2 (2025) → S6W1 (2025) — 7 waves, 3 years |
| **BNPL frequency trend** | S4W2 (2024) → S5W1 (2024) → S5W2 (2025) → S6W1 (2025) — 4 waves, identical wording (for predictive modeling) |
| **BNPL frequency trend (descriptive)** | S3W2 (2023) → S4W1 (2023) → S4W2 (2024) → S5W1 (2024) → S5W2 (2025) → S6W1 (2025) — 5 waves (older wording noted) |
| **Missed payment analysis** | S5W2 + S6W1 — 2 waves with BNPL missed-payment checkbox |
| **Feature selection** | Train set only — weighted statistical screening of covariates (no data leakage) |
| **Predictive modeling (frequency)** | Hurdle model — Stage 1: BNPL adoption (binary), Stage 2: conditional frequency (multiclass) |
| **Predictive modeling (missed payment)** | Logistic Regression + XGBoost + LightGBM (binary, imbalanced) |
| **Segmentation** | K-Means (GMM as optional diagnostic) on BNPL users for latent profile discovery |
| **DL models** | None — data size (~8K rows) is too small for deep learning on tabular data |

### Data Sources
- ZIPs downloaded from `.env` URLs to `data/raw/` using `os.getenv('cfpb_dataset_sample_{n}_url')`
- CSVs extracted from ZIPs in `data/raw/` into DataFrames
- Samples used: S3 (W1+W2), S4 (W1+W2), S5 (W1+W2), S6 (W1 only)
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
- `os` + `dotenv` (`load_dotenv`) — environment variables for dataset download URLs
- `urllib.request` — download ZIP files from CFPB
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

#### 2.2. Download Datasets

Two cells:

**Cell A — Download (run once):**
- Set `project_root = Path.cwd().resolve()`, add to `sys.path`
- `RAW_DIR = project_root / "data" / "raw"`
- Load `.env` via `load_dotenv()`
- Create `data/raw/` directory if missing
- For each sample {3, 4, 5, 6}: check if ZIP exists in `data/raw/`, download from `os.getenv('cfpb_dataset_sample_{n}_url')` if missing using `urllib.request.urlretrieve`
- Prints "[sample]: already exists" or "Downloading..."

**Cell B — Load (run every session):**
- Extract CSVs from ZIPs in `data/raw/`
- Auto-discover CSV name inside each ZIP via `[n for n in zf.namelist() if n.endswith('.csv')][0]`
- Load into DataFrames: `df_s3`, `df_s4`, `df_s5`, `df_s6`
- (No hardcoded `CSV_NAMES` dict — auto-discovery handles varying CFPB filenames)

#### 2.3. Data Mapping & Merging
Map survey columns to canonical variable names. S3W1 only has binary BNPL (`q85`), not the 4-point frequency scale.

| Canonical Name | S3W1 | S3W2 | S4W1 | S4W2 | S5W1 | S5W2 | S6W1 |
|----------------|------|------|------|------|------|------|------|
| `bnpl_adopt` | `q85`>0 | `w2q28`>0 | `q71`>0 | `w2q25`>0 | `q69`>0 | `w2q55`>0 | `q77`>0 |
| `bnpl_freq` | — | `w2q28` | `q71` | `w2q25` | `q69` | `w2q55` | `q77` |
| `missed_bnpl` | — | — | — | — | — | `w2q30h` | `q45h` |
| `id` | `id` | `id` | `id` | `id` | `id` | `id` | `id` |
| `age_bin` | `age_bin` | `age_bin` | `age_bin` | `age_bin` | `age_bin` | `age_bin` | `age_bin` |
| `sex` | `sex` | `sex` | `sex` | `sex` | `sex` | `sex` | `sex` |
| `race` | `race` | `race` | `race` | `race` | `race` | `race` | `race` |
| `education` | `education` | `education` | `education` | `education` | `education` | `education` | `education` |
| `partner` | `partner` | `partner` | `partner` | `partner` | `partner` | `partner` | `partner` |
| `income` | derived | derived | derived | derived | derived | derived | compute from `q10` |
| `fwb` | `fwb` | `fwb` | `fwb` | `fwb` | `fwb` | `fwb` | `fwb` |
| `housing` | `housing` | `housing` | `housing` | `housing` | `housing` | `housing` | `housing` |
| `military` | `military` | `military` | `military` | `military` | `military` | `military` | `military` |
| `q106` | `q106` | `q106` | `q106` | `q106` | `q106` | `q106` | `q106` |
| `q107` | `q107` | `q107` | `q107` | `q107` | `q107` | `q107` | `q107` |
| `financial_stress` | — | — | `q14` | `q12` | `q12` | `q12` | `q45` |

Merge into three DataFrames:

| DataFrame | Waves | Purpose |
|-----------|-------|---------|
| **`df_7wave_adopt`** | S3W1 + S3W2 + S4W1 + S4W2 + S5W1 + S5W2 + S6W1 | **Adoption trend** (7 waves, 3 years: Jan 2022 → Jan 2025) |
| **`df_5wave_freq`** | S3W2 + S4W1 + S4W2 + S5W1 + S5W2 + S6W1 | **Frequency trend** (5 waves, older wording annotation) |
| **`df_4wave_harmonized`** | S4W2 + S5W1 + S5W2 + S6W1 | **Predictive modeling** (identical wording: "four or fewer installments") |

Note: S3W1 only contributes to adoption trend (binary yes/no). It cannot be used for frequency analysis.

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
- Derive `income` for S6W1 from raw `q10` (informational only — **not used in pooled models**)
- Drop `income` from pooled covariate set — 76–83% missing across datasets. Use `fwb` as SES proxy instead.
- Handle missing values in covariates (<7% missing → mode imputation; `bnpl_freq` → logical fillna(0); `weight` → drop row)
- Excluded from imputation: `missed_bnpl` (skip-logic, not imputed), `income` (too sparse)
- No inter-sample dedup (separate samples = separate respondents)
- Record survey weight columns per sample. Two regimes:
  - **Standalone weights** (for §5 descriptive analysis per wave): S3W1=`weight`, S3W2=`w2weight`, S4W1=`weight`, S4W2=`w2weight`, S5W1=`weight`, S5W2=`w2weight_paper`/`w2weight_online`, S6W1=`weight_paper`/`weight_online`. Each wave represents its own population.
  - **Combined weights** (for §6 pooled modeling): Only applies to `df_4wave_harmonized`: S4W2=`w2comb_weight`, S5W1=`comb_weight`, S5W2=`w2comb_weight_paper`/`w2comb_weight_online`, S6W1=`comb_weight_paper`/`comb_weight_online`. The merged dataset is representative as a whole.
- For paper/online splits, select the variant matching the question mode (paper+online or online-only).

### Section 5 — Descriptive Analysis

#### 5.1. BNPL Frequency Distribution
- **5.1.1** Weighted frequencies using standalone weights per wave (see §4 for mapping)
- **5.1.2** Unweighted frequencies for comparison (raw survey counts)

#### 5.2. Trend Analysis

**5.2.1. BNPL Adoption Over Time (7 waves, 3 years)**
- Dataset: `df_7wave_adopt`
- % "Ever used BNPL" plotted across all 7 waves: S3W1 (Jan 2022) through S6W1 (Jan 2025)
- Standalone weights per wave
- Chart features:
  - 7 data points with 95% CIs
  - Year grouping labels (2022: S3W1; 2023: S3W2, S4W1; 2024: S4W2, S5W1; 2025: S5W2, S6W1)
  - Dotted vertical line at S4W2 marking wording change (*"four installments" → "four or fewer installments"*)
- Table: adoption rate, sample size (n), and CI per wave

**5.2.2. BNPL Frequency Category Shifts (5 waves)**
- Dataset: `df_5wave_freq`
- Stacked bar of frequency categories (1-2 / 3-6 / >6 times) per wave
- Standalone weights
- Annotation: "Wording changed at S4W2 from 'four installments' to 'four or fewer installments'"
- Separate mini-chart for `df_4wave_harmonized` (identical wording subset) for clean comparison
- **Known issue — NaN rows silently dropped:** `bnpl_freq` has ~3844 NaN values (respondents who were not asked or did not answer the frequency question). `value_counts(normalize=True)` excludes NaN by default, so "Never" is undercounted because those NaN rows are omitted from the denominator. **Fix:** `fillna(0)` before grouping to treat missing as "Never", then `.reindex(columns=[0,1,2,3], fill_value=0)` to guarantee all 4 categories exist regardless of per-wave presence.
- **Known issue — column count mismatch:** If any wave lacks all 4 frequency categories, `.unstack()` produces fewer columns, causing `.columns = ['Never', '1-2', '3-6', '>6']` to raise `ValueError`. **Fix:** `.reindex()` ensures 4 columns before renaming.

**5.2.3. Missed Payment Rate**
- Dataset: S5W2 + S6W1
- Missed payment rate among BNPL users with 95% CIs
- Standalone weights

#### 5.3. Survey-Weighted Estimates
- Weighted vs unweighted comparison table using standalone weights
- Demonstrate impact of weighting on adoption rate estimates
- **Note:** Same NaN handling as §5.2.2 — `fillna(0)` before grouping, `.reindex()` for column safety

#### 5.4. Feature Selection & Relevance Analysis

**Important:** To prevent data leakage, perform train-test split (80/20, stratified by wave) *before* statistical screening. Run all screening on training set only. Screening is performed on `df_4wave_harmonized` (the predictive modeling dataset).

##### 5.4.1. Train-Test Split
- Split `df_4wave_harmonized` into `train_4wave` and `test_4wave` (80/20, stratified by wave)
- Split `df_2wave` (S5W2+S6W1) into `train_2wave` and `test_2wave` (80/20, stratified by wave)
- Hold test sets aside for Section 6 evaluation

##### 5.4.2. Weighted Statistical Screening
All tests must use survey weights to account for complex sampling design.

| Feature Type | Test | Null Hypothesis | Weighted Variant |
|-------------|------|-----------------|-----------------|
| Continuous (e.g., `fwb`, `age_bin`) | Weighted Kruskal-Wallis H-test | No difference in median across BNPL frequency groups | Apply `weight` as frequency weights |
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
  - `fwb × housing`
  - `sex × partner`
  - `age_bin × fwb` *(optional — check VIF >10 rule)*
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
- **NaN handling:** `dropna(subset=['missed_bnpl'])` on train/test copies — missed payment is only asked of BNPL users; non-users have NaN which must be removed before modeling
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

| Task | Dataset | Baseline | Linear | XGBoost | LightGBM |
|------|---------|----------|--------|---------|----------|
| Stage 1: Adoption | `df_4wave_harmonized` | ✓ | Logistic Regression | ✓ | ✓ |
| Stage 2: Intensity | `df_4wave_harmonized` (users only) | ✓ | Multinomial Logit | ✓ | — |
| Missed Payment | S5W2 + S6W1 | ✓ | Logistic Regression | ✓ | ✓ |

**Total: 8 supervised models** (baselines are floor references, not trained). **+1 unsupervised (K-Means) = 9 total. No DL models.**

**Dataset split rationale:** Predictive modeling (§6) uses `df_4wave_harmonized` (identical wording) for clean coefficients. Trend analysis (§5.2) uses the full 7-wave adoption dataset and 5-wave frequency dataset (with wording annotations) for maximum temporal coverage.

### Section 7 — User Segmentation (Unsupervised)

#### 7.1. Data Preparation
- Subset: BNPL users only (bnpl_freq > 0)
- Features: `bnpl_freq_cond`, `fwb`, `income`, `age_bin`, `missed_bnpl`
- Scale with StandardScaler

#### 7.2. K-Means Clustering
- Grid search k=2..5
- Selection criteria: silhouette score + elbow method (inertia)
- Interpret each cluster profile (mean bnpl_freq, % missed payment, mean fwb, age distribution)
- Label clusters interpretively (e.g., "Strapped heavy users", "Casual healthy users")
- **Optional:** If silhouette < 0.3, run GMM as a diagnostic to check if soft assignment yields better separation.

#### 7.3. Cross-Tabulation
- Cluster × `missed_bnpl`
- Cluster × `bnpl_freq` categories
- Cluster × `fwb` quintiles *(replaces `income` — better coverage)*
- Cluster × age distribution

### Section 8 — Key Insights from Modeling

- **8.1 Model Performance** — which algorithm wins each task?
  - Stage 1 (Adoption): Logistic ROC-AUC vs XGBoost vs LightGBM
  - Stage 2 (Intensity): Multinomial Logit vs XGBoost — accuracy, macro F1, ordinal MAE
  - Missed Payment: Logistic vs XGBoost vs LightGBM — ROC-AUC, PR-AUC, F1
- **8.2 Risk Factors** — who misses payments and why?
  - Missed payment odds ratios from Logistic
  - SHAP beeswarm: top predictors
  - Does bnpl_freq remain significant after controlling for demographics?
  - Interaction effect: age_bin × bnpl_freq
- **8.3 Adoption vs Intensity** — do the drivers differ?
  - SHAP feature rankings from Stage 1 XGBoost vs Stage 2 XGBoost
  - Which features predict "ever trying" vs "using heavily"?
- **8.4 Segments + Trend** — what are the macro patterns?
  - K-Means cluster profiles: number of segments, distinguishing characteristics
  - Adoption trend: accelerating or stabilizing over 3 years?
  - Frequency shift: are heavy users increasing as a share?
  - Adoption growth projection: linear fit over 7 data points, forecast 1-2 periods ahead (illustrative, not rigorous)

### Section 9 — Conclusions and Recommendations

- **9.1** Summary of Findings
- **9.2** Practical Implications (policy, lender risk assessment)
- **9.3** Limitations (survey self-report, 4-point scale granularity, covariate differences across waves, small N for rare events, adoption forecast from 7 irregularly spaced points with wording break — illustrative only)

## Post-Execution Findings & Deviations

The following findings emerged from EDA and were validated against this techspec:

| Finding | Techspec Alignment | Action Taken in Notebook |
|---------|-------------------|-------------------------|
| **Adoption plateau ~30-35%** | Anticipated — hurdle model structure still valid | Added §1 caveat; Stage 1 model retained but noted low ceiling (~70% baseline accuracy) |
| **S3W2→S4W2 jump = wording artifact** | Anticipated (§5.2.1 wording annotation) | Linear forecast capped at 100%; added explicit warning in output |
| **Usage intensity stable ~55/22/23** | Not explicitly flagged in techspec | Added §5.2.2 stability interpretation |
| **Missed payment ~8-9% (rare)** | Anticipated (§6.2 class imbalance) | Confirmed; PR-AUC used as primary metric |
| **Income 76-83% missing** | Anticipated (dropped from pooled features) | Confirmed; FWB used as SES proxy |
| **`df_7wave_adopt` missing from recode loop** | Not anticipated | Added to §4.1 recode loop; §4.3 adoption derivation rewritten to use recoded `bnpl_freq` instead of raw 1-4 `bnpl_adopt` |
| **Model performance weaker than expected** | Not in spec | Added §8.1 and §9.1 notes: Stage 1 max ROC-AUC 0.66, Stage 2 fails to beat baseline, only missed payment shows discriminability (ROC-AUC 0.86) |
| **sklearn version incompatibility** | Not anticipated | Removed `multi_class='multinomial'` from LogisticRegression (deprecated in sklearn ≥1.4) |
| **Comparison tables not rendering** | Not anticipated | Wrapped `pd.DataFrame({...})` with `display()` calls in code_51, code_57, code_64 |

### Implementation Deviations from Techspec (Resolved)

| Techspec Requirement | Notebook Before | Notebook After |
|---------------------|----------------|----------------|
| Weighted feature screening (§5.4.2) | Unweighted chi2/kruskal | Rao-Scott chi-square + Weighted Kruskal-Wallis |
| Survey weights in linear models (§6.1.1) | Not used | `sample_weight` passed to LogisticRegression |
| Cluster-robust SEs by `id` (§6.1.1) | Missing | Added via `statsmodels.Logit` with `cov_type='cluster'` |
| VIF multicollinearity check (§6.1.1) | Missing | Added after each linear model fit |
| GridSearchCV (§6.1.2, §6.1.3, §6.2.3) | Single hardcoded params | Full parameter grids as specified |
| Interaction terms (fwb×housing, sex×partner) | `add_interactions()` defined but not called | Now called before train/test split |

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
| NaN handling | `bnpl_freq` frequency charts: `fillna(0)` before grouping, `.reindex(columns=[0,1,2,3])` to guard against missing categories. Adoption chart: explicit `dropna(subset=['bnpl_adopt'])`. Missed payment modeling: `dropna(subset=['missed_bnpl'])` before split (skip-logic NaN for non-users). |
