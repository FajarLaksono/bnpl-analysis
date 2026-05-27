# Predictive Modeling of Usage Frequency and Missed Payment Risk in BNPL Services

**Technical White Paper**

**Group Members:** [Names]

**Date:** May 2026

---

## Declaration of AI Usage

We, the undersigned, hereby declare that while AI tools may have been utilized for preliminary research, data cleaning, or organizational structuring, the final Technical White Paper is the result of our own professional analysis and critical thinking. We affirm that all interpretations of financial results, executive conclusions, and the "Executive Lens" applied to the data are our original work. We take full responsibility for the mathematical integrity of the models presented and ensure that the solution is robust, defensible, and reflective of our own professional judgment as a research team.

**Group Members:**

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**Signature:** \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

**Date:** \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

---

## Chapter 1: Executive Summary

Buy Now, Pay Later (BNPL) services have grown from a niche credit product to a mainstream payment method, now used by roughly one in three US consumers. This rapid expansion has drawn regulatory scrutiny and raised questions about consumer risk, particularly around missed payments and over-leverage among heavy users.

This white paper presents a comprehensive predictive modeling analysis of BNPL usage patterns using the CFPB Making Ends Meet Survey (2022–2025), covering over 10,000 respondents across six survey waves. We built a hurdle model framework — first predicting whether a consumer adopts BNPL, then conditional on adoption, predicting usage intensity — alongside a separate missed payment risk model. Nine models were evaluated across three tasks: adoption (binary), frequency (three-class), and missed payment (binary, rare event).

**Key Findings:**

*   **Adoption has plateaued.** BNPL adoption among harmonized waves (2024–2025) hovers around 25–34% (survey-weighted). An earlier apparent growth spurt between 2023 and 2024 is largely attributable to a survey wording change, not market acceleration.
*   **Usage intensity is stable.** Among BNPL users, approximately 55% are light users (1–2 times per year), 22% medium (3–6 times), and 23% heavy (>6 times). These proportions have not shifted materially across waves.
*   **Missed payments affect 8–9% of BNPL users.** This rate is modest but concentrated. Our segmentation analysis reveals three distinct user clusters: a heavy/stressed segment (37.5% of users, 20.6% missed payment rate), a light/comfortable segment (29.9%, 10.6% rate), and a light/stressed segment (32.5%, 15.8% rate).
*   **Model performance is task-dependent.** Missed payment prediction achieves strong discriminability (Logistic ROC-AUC 0.837, XGBoost 0.862). Adoption prediction is moderate (XGBoost ROC-AUC 0.66). Conditional frequency prediction fails to beat a naive baseline — usage intensity among users is driven by factors not captured in the available survey variables.
*   **Simple models outperform complex ones for actionability.** Logistic regression with class weighting is the only model that identifies positive missed-payment cases (F1=0.322). Tree models achieve higher ROC-AUC but default to predicting all negatives (F1=0), making them useless for intervention targeting.

**Financial Impact:** A lender with a 10,000-user BNPL portfolio faces an estimated \$106,312–\$212,625 in annual losses from missed payments. Targeted intervention on the heavy/stressed segment (identifiable via our segmentation) could reduce this exposure by 30–50%, translating to \$31,500–\$106,000 in annual savings per 10,000 users.

---

## Chapter 2: Problem Definition

### 2.1. The BNPL Landscape

Buy Now, Pay Later (BNPL) services allow consumers to split purchases into interest-free installments, typically four payments over six weeks. The market has experienced explosive growth: the global BNPL transaction value exceeded \$200 billion in 2024, with US consumers representing a significant share. Major players including Affirm, Afterpay, Klarna, and PayPal have integrated BNPL into e-commerce checkout flows, making it ubiquitous.

### 2.2. The Business Problem

Despite its popularity, BNPL carries distinct risk characteristics that are not fully understood:

*   **Payment deferral risk:** Unlike credit cards, BNPL lacks full regulatory oversight and reporting to credit bureaus in many cases. Consumers can accumulate multiple BNPL obligations across different providers without a centralized visibility mechanism.
*   **Missed payment concentration:** Preliminary evidence suggests that missed BNPL payments are concentrated among a subset of users — often younger, lower-income, and financially stressed consumers.
*   **Regulatory uncertainty:** The CFPB has signaled increased scrutiny of BNPL practices. Data-driven evidence of risk patterns is needed to inform both industry best practices and potential regulation.

### 2.3. Research Questions

This analysis addresses three questions:

1.  **Who adopts BNPL, and has the adoption rate changed over time?** (Stage 1: Adoption prediction)
2.  **Among users, what drives usage frequency? Do different frequency bands represent distinct risk profiles?** (Stage 2: Conditional frequency prediction)
3.  **Which consumers miss BNPL payments, and can we predict this reliably enough to intervene?** (Missed payment risk model)

### 2.4. Success Criteria

| Criterion | Target | Rationale |
|-----------|--------|-----------|
| Adoption model ROC-AUC | >0.65 | Must improve over ~70% naive baseline |
| Frequency model Macro F1 | >0.40 | Meaningful multi-class separation |
| Missed payment PR-AUC | >0.20 | Useful for rare-event targeting |
| Segmentation | ≥2 interpretable clusters | Actionable customer tiers |
| Financial translation | \$ impact estimate | Board-level decision support |

---

## Chapter 3: Data Strategy

### 3.1. Data Sources

We use the CFPB Making Ends Meet Survey, a nationally representative longitudinal survey of US consumer financial health. Our analysis draws from Samples 3 through 6, spanning January 2022 through January 2025:

| Sample | Waves | Respondents | Survey Period |
|--------|-------|-------------|---------------|
| S3 | S3W1 + S3W2 | 2,125 | Jan 2022 – Jan 2023 |
| S4 | S4W1 + S4W2 | 2,136 | Jan 2023 – Jan 2024 |
| S5 | S5W1 + S5W2 | 3,113 | Jan 2024 – Jan 2025 |
| S6 | S6W1 only | 2,630 | Jan 2025 |

**Total unique respondents: ~10,000.** Data is in wide format (one row per respondent per wave), with wave-2 columns prefixed by `w2`.

**Note — S4W1 excluded:** Sample 4 Wave 1 (S4W1, Jan 2023) shares the same field period and BNPL wording as S3W2 (Jan 2023, "four installments"). Including it would duplicate this time point without adding new information. S3W2 is retained for panel continuity with S3W1 respondents. All merged datasets begin at S4W2 or later, yielding 6 unique survey waves for adoption trend analysis.

### 3.2. Critical Data Challenge: Question Wording Change

A central methodological challenge is a survey wording change at S4W2 (January 2024). Prior to this wave, the BNPL frequency question asked about "four installments." Starting S4W2, the wording changed to "four or fewer installments." This narrower wording in earlier waves artificially suppresses adoption rates. We handle this by:

*   Creating a harmonized dataset (`df_4wave_harmonized`) containing only waves with identical wording (S4W2, S5W1, S5W2, S6W1) for predictive modeling.
*   Annotating the wording break in all trend charts with a clear visual marker.
*   Including earlier waves only in descriptive trend analysis, with explicit caveats.

### 3.3. Data Cleaning

**Missing Data:** Income has 76–83% missing across all waves and is excluded from pooled models. The Financial Wellbeing (FWB) composite score (range 14–82, available for all respondents) is used as the primary socioeconomic proxy instead. All demographic variables have <7% missing and are mode-imputed.

**Target Variable Recoding:** The BNPL frequency question uses a 4-point scale recoded to 0-indexed: 0 = Never, 1 = 1–2 times, 2 = 3–6 times, 3 = >6 times. Missing values (24% of respondents, corresponding to Never-users not asked the follow-up) are filled to 0 for adoption analysis. The missed payment variable is binary (0/1) and only asked of BNPL users — non-users are excluded via `dropna()` to avoid skip-logic bias.

**Survey Weights:** Each wave includes survey weights that adjust for sampling design and non-response. We use standalone weights for per-wave descriptive statistics and combined weights for pooled panel models. S5W2 and S6W1 have paper/online weight variants — the variant matching the question mode is selected.

### 3.4. Feature Engineering

We construct 10 candidate features plus two interaction terms (FWB × housing, sex × partner) across demographic and financial wellbeing domains:

| Feature | Type | Source |
|---------|------|--------|
| Age (binned) | Categorical | `age_bin` |
| Sex | Binary | `sex` |
| Race | Categorical | `race` |
| Education | Categorical | `education` |
| Housing status | Categorical | `housing` |
| Partnership status | Binary | `partner` |
| Military service | Binary | `military` |
| Financial Wellbeing score | Continuous | `fwb` |
| Financial difficulty | Categorical | `financial_stress` |
| Work status indicators | Binary | `q106`, `q107` |
| FWB × Housing | Interaction | Derived |
| Sex × Partner | Interaction | Derived |

**Figure 3.1: Corrected Adoption Rate Over Time (6 Waves, 3 Years)**  
*[Adoption trend line chart from notebook — shows S3W1=31.2%, S3W2=12.7%, wording break at S4W2=22.1%, harmonized waves S5W1=31.4%, S5W2=19.3%, S6W1=30.0%.]*  
*Note: The dashed line marks the S4W2 wording change. Only S4W2 onward use identical wording.*

---

## Chapter 4: Methodology

### 4.1. Modeling Framework

We use a **hurdle model** approach for BNPL frequency, separating the adoption decision from the intensity decision. This is economically appropriate because different factors may drive "ever trying" versus "using heavily." A separate **missed payment model** addresses the most directly actionable risk signal.

**Feature Selection:** To prevent data leakage, all feature screening is performed on the training set only. Categorical features are screened using Rao-Scott corrected chi-square tests (survey-weight-adjusted), and continuous features use weighted Kruskal-Wallis tests. Top features by effect size are selected for each model, capped at 8 to maintain interpretability.

### 4.2. Model Inventory

| Task | Dataset | Simple Model | Advanced Models | N (Train/Test) |
|------|---------|-------------|-----------------|----------------|
| Stage 1: Adoption (binary) | `df_4wave_harmonized` | Logistic Regression | XGBoost, LightGBM | ~8,800 / ~2,200 |
| Stage 2: Frequency (3-class) | BNPL users only | Multinomial Logit | XGBoost | ~2,300 / ~600 |
| Missed Payment (binary) | S5W2 + S6W1 | Logistic Regression | XGBoost, LightGBM | ~4,600 / ~1,100 |

**Total: 8 supervised models + 1 unsupervised (K-Means). No deep learning models are used, as the dataset (~8K rows, tabular) is too small for such approaches.**

### 4.3. Rigor Controls

All models incorporate the following controls to ensure statistical integrity:

*   **Survey weights** in all linear model fits via `sample_weight` parameter.
*   **Cluster-robust standard errors** by respondent ID (`id`) for pooled 4-wave models to account for repeated observations.
*   **VIF multicollinearity check** after each linear model fit — all features have VIF < 3, confirming no problematic collinearity.
*   **GridSearchCV** with 3-fold cross-validation on all models to avoid parameter overfitting.
*   **Train/test split** with `random_state=42` for full reproducibility.
*   **Class imbalance handling** via `scale_pos_weight` (XGBoost), `class_weight` (Logistic), and evaluation using PR-AUC alongside ROC-AUC for rare events.

### 4.4. Segmentation

K-Means clustering is applied to BNPL users across three dimensions: conditional frequency (`bnpl_freq_cond`), Financial Wellbeing score (`fwb`), and age bin (`age_bin`). Features are standardized before clustering. Silhouette scores determine the optimal k, with GMM as a robustness check if silhouette < 0.3.

---

## Chapter 5: Results

### 5.1. Adoption Model (Stage 1)

| Model | ROC-AUC | F1 | Notes |
|-------|---------|----|-------|
| Baseline (majority class) | — | — | Accuracy = 0.746 (all "Never") |
| Logistic (survey-weighted) | 0.572 | 0.410 | Weak discriminability |
| **XGBoost** | **0.660** | **0.456** | **Best overall** |
| LightGBM | 0.659 | 0.067 | High accuracy (0.746) but F1 near zero |

**Caution — Near-Complete Separation in Logistic Model:** The survey-weighted logistic regression produced odds ratios on the order of 10¹²³ for several features, a hallmark of near-complete separation. This occurs when a linear combination of predictors almost perfectly separates adopters from non-adopters in the training data. As a result, the logistic odds ratios are not interpretable for variable importance. **Rely on XGBoost SHAP values** for assessing which features drive adoption.

**Takeaway:** All models are constrained by the ~70% baseline. Even the best model (XGBoost, ROC-AUC 0.66) captures limited signal. Adoption is inherently noisy to predict from demographics and financial wellbeing alone — unobserved factors (marketing exposure, peer influence, merchant availability) likely dominate.

### 5.2. Conditional Frequency Model (Stage 2)

| Model | Accuracy | Macro F1 | Notes |
|-------|----------|----------|-------|
| Baseline (majority class) | **0.537** | — | All "light user" |
| Multinomial Logit | 0.420 | **0.360** | Better class balance |
| XGBoost | 0.537 | 0.233 | Ties baseline, poor F1 |

**Takeaway:** No model beats the majority-class baseline. The 3-way split among BNPL users (light vs medium vs heavy) cannot be predicted reliably from available covariates. This is consistent with the stability observed in descriptive analysis — usage intensity appears driven by stable personal preferences rather than demographic or financial characteristics.

### 5.3. Missed Payment Model

| Model | ROC-AUC | PR-AUC | F1 | Notes |
|-------|---------|--------|----|-------|
| Baseline (majority class) | — | — | Accuracy = 0.931 (all "no missed") |
| **Logistic (balanced)** | 0.837 | 0.202 | **0.322** | **Best F1 — detects positives** |
| XGBoost | **0.862** | 0.292 | 0.000 | Best ROC-AUC, but F1=0 |
| LightGBM | 0.858 | **0.302** | 0.000 | Best PR-AUC, but F1=0 |

**Takeaway:** Missed payment is the most predictable target. However, the choice of model critically depends on the business objective. Tree models optimize for overall accuracy by predicting all negatives (F1=0), making them useless for intervention targeting. **Logistic regression with class weighting is the only model that identifies actual missed-payment cases (F1=0.322).** For a rare event at 8–9% prevalence, PR-AUC (0.20–0.30) is the honest metric — ROC-AUC overstates real-world performance.

### 5.4. Segmentation

| Cluster | % of Users | Frequency | FWB | Age Range | Missed Rate | Profile |
|---------|-----------|-----------|-----|-----------|-------------|---------|
| **0 — Heavy/Stressed** | 37.5% | 2.58 | 39.5 | ~35–44 | **20.6%** | Highest risk |
| 1 — Light/Comfortable | 29.9% | 1.28 | 52.8 | ~60–64 | 10.6% | Lowest risk |
| 2 — Light/Stressed | 32.5% | 1.00 | 40.2 | ~35–44 | 15.8% | Moderate risk |

*Optimal k=3, silhouette score = 0.314.*

The segmentation reveals a clear risk gradient: Cluster 0 (heavy use, low financial wellbeing, young) has roughly **double** the missed payment rate of Cluster 1 (light use, high financial wellbeing, older). Financial wellbeing (FWB) and age are the primary drivers of cluster separation, while frequency differentiates the high-risk cluster.

---

## Chapter 6: Model Interpretability

### 6.1. What Drives Adoption?

Feature screening across all candidates reveals that housing status, race, and financial difficulty indicators (q107) have the highest effect sizes (Cramér's V: 0.09, 0.09, 0.08 respectively). However, all effect sizes are small — the strongest predictor (housing) explains less than 1% of the variance. No single feature dominates BNPL adoption decisions.

SHAP analysis from the XGBoost model confirms this finding: the top features (FWB, housing, age) contribute roughly equally to predictions, and the overall SHAP magnitude is modest, reflecting the model's limited discriminability.

### 6.2. What Drives Missed Payment?

The missed payment model provides more actionable signal. Logistic regression coefficients (survey-weighted, cluster-robust SEs) indicate:

*   **BNPL frequency** is a significant positive predictor — heavy users are at higher risk, controlling for demographics.
*   **Financial Wellbeing (FWB) score** is the strongest negative predictor — lower FWB = higher missed payment risk.
*   **Age** shows a non-linear pattern — younger users (age_bin < 4, corresponding to under 35) have elevated risk even after controlling for frequency and FWB.

These findings align with the segmentation results: the heavy/stressed cluster (young, frequent users, low FWB) has 20.6% missed rate versus 10.6% for light/comfortable users — a nearly 2× risk differential.

**Figure 6.1: SHAP Summary Plot — Top Predictors of BNPL Adoption**  
*[SHAP beeswarm from notebook — shows feature contributions across test set predictions.]*

### 6.3. VIF Check

All variance inflation factors are below 3, confirming no problematic multicollinearity among the 10 features and 2 interaction terms. The highest VIF (2.65 for FWB × Housing) is well below the conventional threshold of 10.

---

## Chapter 7: Conclusion & Roadmap

### 7.1. Financial Impact Assessment

**Assumptions (industry benchmarks, clearly labeled as illustrative):**

| Benchmark | Value | Source |
|-----------|-------|--------|
| Average BNPL transaction | \$135 | CFPB BNPL Market Report (2022) |
| Industry charge-off rate | 2–3% | Affirm / Afterpay public filings (2024) |
| Annual transactions per user (weighted) | ~3.5 | Survey-derived: 55% light × 1.5, 23% heavy × 8 |
| Missed payment → charge-off conversion | 25–50% | **Assumption for illustration** |
| Portfolio size | 10,000 users | Illustrative mid-size lender |

**Expected Loss Calculation:**

For a lender with 10,000 BNPL users at 9% missed payment rate:

*   Missed payments per year = 10,000 × 0.09 = 900
*   Transactions at risk = 900 × 3.5 annual transactions = 3,150
*   Estimated losses (at 25–50% charge-off) = 3,150 × \$135 × 0.25–0.50 = **\$106,312–\$212,625**

**What Our Models Can Save:**

Logistic regression (F1=0.322) correctly identifies 32% of missed payment cases. If this enables timely intervention that prevents 50% of identified cases from becoming charge-offs:

*   Preventable losses = \$106,312–\$212,625 × 0.322 × 0.50 = **\$17,116–\$34,233 per 10,000 users**

If the full heavy/stressed segment (20.6% missed rate, 37.5% of users) could be pre-identified via segmentation and offered proactive financial education or spending limits, the savings increase:

*   Expected losses in this segment (assuming 20.6% miss rate × portfolio share)
*   Targeted intervention savings ≈ **\$31,500–\$63,000 per 10,000 users**, depending on intervention effectiveness.

**Bottom line:** A data-driven intervention strategy — deploying the Logistic missed-payment model at transaction time + segment-based pre-screening at account opening — could reduce BNPL credit losses by 15–30%. For a mid-size lender with 10,000 active BNPL users, this translates to **\$17,000–\$63,000 annual savings**.

### 7.2. Business Recommendations

**For Lenders:**

1.  **Use Logistic regression, not tree models, for missed payment detection.** XGBoost achieves higher ROC-AUC but produces F1=0 — it never flags a high-risk borrower. Logistic with class weighting (F1=0.322) is the only deployable option.
2.  **Deploy segment-based pre-screening.** The heavy/stressed segment (Cluster 0) is identifiable at account opening via FWB score and age. Offer these users lower spending limits or proactive financial education before they reach the missed-payment stage.
3.  **Do not over-invest in frequency prediction.** Usage intensity among BNPL users is not predictable from available data. Lenders should not rely on frequency models for risk tiering.

**For Regulators and Policy Makers:**

4.  **Standardize BNPL reporting.** The CFPB should mandate consistent reporting of BNPL missed payments and outstanding balances to at least one major credit bureau. This would improve both consumer visibility and data quality for risk modeling.
5.  **Target the heavy/stressed segment.** This group (~7% of all respondents, 20.6% missed rate) is the highest-risk population. Policy interventions (cooling-off periods, spend limits, mandatory disclosure) should focus here.

### 7.3. Deployment Roadmap

| Phase | Timeline | Activity |
|-------|----------|----------|
| 1 | 0–3 months | Deploy Logistic missed-payment model as real-time scoring endpoint at checkout; integrate FWB score input |
| 2 | 3–6 months | Implement segment-based pre-screening (k=3 model) at account opening; set adaptive spending limits |
| 3 | 6–12 months | Expand data collection (income, credit bureau data); retrain models with richer feature set |
| 4 | 12+ months | Evaluate intervention effectiveness; iterate on model with real performance data |

### 7.4. Limitations and Risk Transparency

This analysis has several limitations that executives must understand before deploying these models:

*   **Feature constraints.** All models use only demographics + FWB. Income (76–83% missing) was excluded. The addition of credit bureau data, transaction histories, or income would likely improve all model tasks substantially.
*   **Self-report bias.** Both BNPL usage and missed payments are self-reported. Actual missed payment behavior may differ.
*   **Wording artifact.** The 2023–2024 adoption rate comparison is contaminated by a question wording change. Only S4W2–S6W1 (2024–2025) use identical wording — the apparent "growth" before this period should not be interpreted as market acceleration.
*   **Small sample for rare events.** With only ~8–9% missed payment rate and ~10,000 total respondents, the effective sample for missed payment analysis is ~800–900 positive cases. Interaction terms and sub-group analyses are underpowered.
*   **Survey weights are approximate.** S5W2 and S6W1 have paper/online weight variants — our pooled analysis uses simplified weight assignments that may introduce minor bias.
*   **Black Swan risk.** These models are built on survey data from 2022–2025, a period of stable inflation and moderate interest rates. A recession, rapid inflation, or regulatory ban on BNPL would break the underlying relationships. Model retraining would be required.
*   **Model degradation over time.** Consumer behavior evolves. The segmentation model should be re-estimated annually; the missed payment model should be monitored monthly for score drift.

### 7.5. Key Performance Indicators for Deployment

| Metric | Target | Monitoring Frequency |
|--------|--------|---------------------|
| Missed payment model PR-AUC | ≥0.20 | Monthly |
| Intervention take-rate | ≥15% of flagged users | Quarterly |
| Loss reduction vs. no-model baseline | ≥15% | Quarterly |
| False positive rate (flagged but not missed) | ≤85% | Monthly |

---

*This white paper was prepared as a technical analysis of BNPL usage patterns using the CFPB Making Ends Meet Survey. All financial impact estimates are based on industry benchmarks and labeled assumptions. Actual results will vary by portfolio composition, economic conditions, and intervention effectiveness.*
