# CFPB Making Ends Meet Survey — Complete Feature Dictionary

## Overview

This document catalogs all features across the CFPB Making Ends Meet survey datasets used in this project. It covers **1,073 unique variable names** across 4 samples (S3, S4, S5, S6), organized into 23 thematic categories.

For BNPL-specific column mapping and cross-wave harmonization, see `docs/techspec/bnpl-datasets.md`. For the analysis plan, see `docs/techspec/bnpl-usage-frequency-analysis.md`.

---

## Dataset Overview

| Sample | Waves | Field Period | Columns | Rows | CSV File (inside ZIP) |
|--------|-------|-------------|---------|------|-----------------------|
| **S3** | S3W1 + S3W2 | Jan 2022 & Jan 2023 | 404 | 2,125 | `MEM_S3W1W2_PUF.csv` |
| **S4** | S4W1 + S4W2 | Jan 2023 & Jan 2024 | 396 | 2,136 | `MEM_S4W1W2_PUF.csv` |
| **S5** | S5W1 + S5W2 | Jan 2024 & Jan 2025 | 491 | 3,113 | `MEM_S5W1W2_PUF.csv` |
| **S6** | S6W1 only | Jan 2025 | 346 | 2,630 | `MEM_S6W_PUF.csv` |

**File locations:** `docs/cfpb_making-ends-meet_data-sample-{3,4,5,6}.zip`

---

## 1. Survey Administration & Weights (13 vars)

These appear in **all** samples unless noted.

| Column | Description | Samples | Type |
|--------|------------|---------|------|
| `ID` | Random consumer identifier | S3,S4,S5,S6 | Numeric |
| `weight` | Cross-sectional weight for Wave 1 analysis | S3,S4,S5 | Numeric |
| `w2weight` | Panel weight for Wave 1+2 analysis (addresses attrition) | S3,S4 | Numeric |
| `w2comb_weight` | Combined weight for cross-sectional analysis with next sample's W1 | S3,S4 | Numeric |
| `comb_weight` | Combined weight for cross-sectional analysis with prior sample's W2 | S4,S5 | Numeric |
| `weight_paper` | Weight for S6W1 questions on paper+online | S6 | Numeric |
| `weight_online` | Weight for S6W1 online-only questions | S6 | Numeric |
| `comb_weight_paper` | Combined weight S6W1+S5W2 for paper variables | S6 | Numeric |
| `comb_weight_online` | Combined weight S6W1+S5W2 for online variables | S6 | Numeric |
| `w2weight_paper` | Panel weight S5W1+W2 for paper+online questions | S5 | Numeric |
| `w2weight_online` | Panel weight S5W1+W2 for online-only questions | S5 | Numeric |
| `w2comb_weight_online` | Combined weight S6W1+S5W2 online analysis | S5 | Numeric |
| `w2comb_weight_paper` | Combined weight S6W1+S5W2 paper analysis | S5 | Numeric |

---

## 2. Financial Wellbeing & Self-Assessment (11 vars)

### Core FWB Score
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `fwb` | Financial well-being score (computed from q1a–q1e + q2a–q2b) | Continuous, ~14–82 (higher = better) | S3,S4,S5,S6 |
| `w2fwb` | Financial well-being score, Wave 2 | Same scale | S3,S4,S5 |

### Financial Self-Assessment (q1a–q1e)
Scale: **1**=Completely → **5**=Not at all

| Column | Description | Samples |
|--------|------------|---------|
| `q1a` | I know how to make complex financial decisions | S3,S4,S5,S6 |
| `q1b` | I am comfortable using English to perform financial transactions | S5,S6 |
| `q1c` | I am just getting by financially | S3,S4,S5,S6 |
| `q1d` | I am concerned that the money I have or will save won't last | S3,S4,S5,S6 |
| `q1e` | Because of my money situation, I feel I will never have the things I want | S4,S5,S6 |

### Financial Behavior (q2a–q2b)
Scale: **1**=Always → **5**=Never

| Column | Description | Samples |
|--------|------------|---------|
| `q2a` | I have money left over at the end of the month | S3,S4,S5,S6 |
| `q2b` | My finances control my life | S3,S4,S5,S6 |

### Wave 2 Versions
| Column | Description | Scale | Samples |
|--------|------------|-------|---------|
| `w2q1a` | I am comfortable using English (W2) | 1–5 | S3 |
| `w2q1b` | I am just getting by financially (W2) | 1–5 | S3 |
| `w2q1c` | I am concerned money won't last (W2) | 1–5 | S3 |
| `w2q1d` | Because of money, will never have things I want (W2) | 1–5 | S3 |
| `w2q2a` | Money left over at end of month (W2) | 1–5 (Always–Never) | S3,S4,S5 |
| `w2q2b` | Finances control my life (W2) | 1–5 (Always–Never) | S3,S4,S5 |

---

## 3. Employment & Work Status (17 vars)

### Your Work Status (q3a1–q3a7)
Check-all-that-apply: **0**=Not Selected, **1**=Selected. Present in **all** samples.

| Column | Label |
|--------|-------|
| `q3a1` | Self-employed |
| `q3a2` | Work full time |
| `q3a3` | Work part time |
| `q3a4` | Retired |
| `q3a5` | Temporarily laid off or on leave |
| `q3a6` | Unemployed |
| `q3a7` | Not working for pay |

### Spouse/Partner Work Status (q3b1–q3b7)
Same coding and availability.

| Column | Label |
|--------|-------|
| `q3b1` | Self-employed |
| `q3b2` | Work full time |
| `q3b3` | Work part time |
| `q3b4` | Retired |
| `q3b5` | Temporarily laid off or on leave |
| `q3b6` | Unemployed |
| `q3b7` | Not working for pay |

### Additional Employment Variables
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q4` | Past 12 months: Frequency working from home | 1=Most days, 2=A few times a month, 3=A few times a year, 4=Never/rarely | S3,S4,S5,S6 |
| `w2q4` | Do you own your own business? | 0=No, 1=Yes | S3,S4,S5 |
| `w2q3a1–w2q3a7` | Wave 2 work status (same as q3a) | 0/1 | S3,S4,S5 |
| `w2q3b1–w2q3b7` | Wave 2 spouse/partner work status | 0/1 | S3,S4,S5 |

---

## 4. Demographics & Household Composition (25 vars)

### Computed Demographics (appear in ALL samples)
| Column | Description | Values |
|--------|------------|--------|
| `race` | Race/ethnicity (mutually exclusive, per FDIC coding) | 1=White, 2=Black, 3=Hispanic, 4=Asian, 5=Other |
| `sex` | Gender | 1=Male, 2=Female |
| `age_bin` | Age (binned) | 0=18–24, 1=25–29, 2=30–34, 3=35–39, 4=40–44, 5=45–49, 6=50–54, 7=55–59, 8=60–61, 9=62–64, 10=65–69, 11=70–74, 12=75–79, 13=80+ |
| `education` | Education (recoded) | 1=High school or less, 2=Some college no degree, 3=Two-year/vocational, 4=College/postgrad |
| `partner` | Married or living with a partner | 0=No, 1=Yes |
| `military` | Military service (excludes military spouses) | 0=None, 1=Some service |
| `housing` | Housing status (recoded) | 1=Homeowner, 2=Renter, 3=Neither |
| `w2housing` | Wave 2 housing status (S4 only) | Same coding |

### Marital Status (check-all-that-apply)
S3 uses `q110a–q110f`; S4,S5,S6 use `q105a–q105f`. **0**=Not Selected, **1**=Selected.

| Column (S3) | Column (S4,S5,S6) | Label |
|-------------|--------------------|-------|
| `q110a` | `q105a` | Married |
| `q110b` | `q105b` | Living with a partner / Never married, living with partner |
| `q110c` | `q105c` | Never married / Never married, not living with partner |
| `q110d` | `q105d` | Separated |
| `q110e` | `q105e` | Divorced |
| `q110f` | `q105f` | Widowed |

### Household Composition
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q102` (S3,S5,S6) / `q106` (S4) | Number of other adults in HH besides self & spouse | 1=None, 2=1 other, 3=2 or more | All |
| `q103` (S3,S4,S5) / `q107` (S4,S5,S6) | Number of children in HH | 1=None, 2=1, 3=2 or more | All |
| `q104` | Is English your preferred language? | 0=No, 1=Yes | S3,S5,S6 |
| `q111` | Highest level of education (detailed) | 1=Less than HS, 2=HS, 3=Technical/vocational, 4=Some college, 5=Two-year, 6=Four-year, 7=Some grad school, 8=Grad/professional | S6 |
| `q112` | Currently attending school? | 1=Yes full time, 2=Yes part time, 3=No | S6 |
| `q113` | Parent completed four-year college degree? | 0=No, 1=Yes | S6 |

---

## 5. Income, Finances & Expenses (37 vars)

### Income Groups (computed)
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `income` | Income group (recoded) | 1=$20k or less, 2=$20–50k, 3=$50–80k, 4=$80–125k, 5=$125k+ | All |
| `w2income` | Wave 2 income group | Same | S3,S4 |

### Assets & Banking
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q5` | HH checking/savings account balance | Binned | All |
| `q6` | HH currently has a checking account? | 0/1 | All |
| `q7` | HH currently has a savings account? | 0/1 | All |
| `q8` | HH currently has a prepaid card? | 0/1 | All |
| `q9` | HH currently uses a money order? | 0/1 | All |
| `q10` | HH currently uses a check-cashing service? | 0/1 | S3,S4,S5 |
| `q11` | HH currently uses a payday loan? | 0/1 | All |
| `q12` | HH currently uses a pawn shop? | 0/1 | All |
| `q15` | Total money in checking + savings now | Bands | All |
| `q16` | Monthly non-retirement saving habits | 1=Saves regularly, 2=Saves what's left, 3=Spends more than income, etc. | All |
| `q17` | HH has for unexpected expenses (non-retirement) | Bands | All |
| `q18` | In past year, HH's total income (binned) | Bands | S5,S6 |
| `q19` | Frequency of worry about money | 1=Never → 5=Always | All |
| `q20` | HH's typical monthly spending on food/meals | Numeric | S3,S4,S5,S6 |
| `q21` | HH spending on non-food regular expenses | Numeric | S3,S4,S5,S6 |
| `q22` | HH debt payments per month | Numeric | S3,S4,S5,S6 |

### Wave 2 Income/Expense Questions
| Column | Description | Samples |
|--------|------------|---------|
| `w2q5` | Monthly rent/mortgage spending | S3,S4,S5 |
| `w2q6` | Since Jan 2022, how have normal HH expenses changed? | S3,S4,S5 |
| `w2q7` | In last year, how has amount of money HH has left after expenses changed? | S3,S4,S5 |
| `w2q8` | Annual gross income in 2022 (11 bands) | S3,S4,S5 |
| `w2q10` (S3) / `w2q11` (S4,S5) | Income varies month to month? / Which describes HH income? | S3,S4,S5 |
| `w2q11` (S3) / `w2q12` (S4,S5) | If HH lost main income, how long could it cover expenses? | S3,S4,S5 |

### S6 Income/Expenses (revised numbering)
| Column | Description |
|--------|------------|
| `q10` | HH total income in past year (11 raw bands); mapped to 5-group `income` | S6 only |
| `q18a–q18g` | HH currently has: checking, savings, prepaid card, money order, check cashing, payday, pawn? |
| `q21a–q21f` | HH owes money on: credit card, auto loan, student loan, medical, personal loan, family loan |

---

## 6. Housing & Living Situation (20 vars)

| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q26` | How many months did the most recent difficulty last? | Numeric | All |
| `q27` | Do you or someone in HH own your current residence? | 0=No, 1=Yes | S4,S5,S6 |
| `q28` | Monthly rent/mortgage payment including taxes/insurance | Numeric (rounded) | S3,S5,S6 |
| `q29` | Do you or someone in HH own current residence? | 0/1 | S3,S4,S5 |
| `q30` | When did you move to current residence? | 1=<1 year, 2=1-2 years, 3=3-5 years, 4=6+ years | All |
| `q31` | Number of times moved in past 5 years | 0=None, 1=1 time, 2=2 times, 3=3+ times | All |
| `q32` | Past year: How often not paid/late with rent? | 1=Never, 2=Once, 3=More than once | S3,S6 |

### Housing Events
| Column | Description | Samples |
|--------|------------|---------|
| `q28a–q28d` | Past year: threatened eviction, eviction notice, moved rent increase, lease not renewed | S4 |
| `q29a,q29c,q29d,q29e` | Past year: threatened eviction, moved rent increase, lease not renewed, rent increase strained budget | S6 |
| `q32a` | Past year: Threatened with eviction | S5 |

---

## 7. Government Benefits & Assistance (6 vars)

| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `w2q9a` | HH received TANF benefits | 0=No, 1=Yes | S3 |
| `w2q9b` | HH received SNAP benefits | 0/1 | S3 |
| `w2q9c` | HH received EITC | 0/1 | S3 |
| `w2q9d` | HH received WIC | 0/1 | S3 |
| `w2q9e` | HH received SSI or SSDI | 0/1 | S3 |
| `w2q9f` | HH received LIHEAP | 0/1 | S3 |

S4,S5 use a combined check-all `w2q9` variable instead.

---

## 8. Healthcare, Insurance & Medical Debt (55 vars)

### Health Insurance
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q47` / `q43` | Has health insurance | 0=No, 1=Yes | S3,S4,S5,S6 / S4,S5 |
| `q48` | Covered by Medicaid/Medical Assistance | 0/1 | S3,S4,S5,S6 |
| `q49` | Everyone else in HH has health insurance | 0/1 | S3,S4,S5,S6 |
| `q100` / `q101` | Do you / everyone in HH have health insurance | 0/1 | S3,S4,S5,S6 |
| `q108` | Children covered by Medicaid/CHIP | 0=No, 1=Yes | S6 |
| `q104` (S6) | Self-rated health | 1=Excellent → 5=Poor | S6 |

### Medical Debt
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q50` | Past-due medical/dental bills? | 0=No, 1=Yes | All |
| `q51` | Medical/dental bills paying off over time to provider? | 0/1 | All |
| `q52` | Owe money because took out loan/used CC for medical bills? | 0/1 | All |
| `q53` | Has medical credit card? | 0/1 | All |
| `q54` | Contacted by third-party collector for medical debt? | 0/1 | All |
| `q55` | Number of medical bill collections | 1=1 bill, 2=2-4, 3=5+ | All |
| `q56` | Frequency of medical collection attempts | 1=>weekly → 6=only once | All |
| `q57` | Amount in past-due medical/dental bills being collected | Numeric | All |
| `q58` | Disputed most recent medical collection? | 0=No, 1=Yes | All |
| `q59` | Approximate credit score | Numeric (300-850) | All |

### Wave 2 Medical (S3,S4,S5)
Detailed medical debt collection module: `w2q32`–`w2q56f` covering insurance, collections, lawsuits, disputes, and payment methods.

---

## 9. Credit Cards & Account Management (20 vars)

| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q95` / `w2q25` / `q81` | Do you currently have a credit card? | 0=No, 1=Yes | All |
| `q96` / `w2q26` / `q82` | Unpaid balance after last payment? | 0=No, 1=Yes | All |
| `q84` | Expect to pay full balance in next year? | 0=No, 1=Yes | S3,S5,S6 |
| `q85` | Has credit card autopay? | 0=No, 1=Yes | S3,S5,S6 |
| `q86` | Credit card autopay amount setting | 1=Minimum, 2=Full statement, 3=Some other | S3,S5,S6 |
| `w2q27` / `[S5,S6]q87` | Past 12 months: incurred late fee on credit card? | 0=No, 1=Yes | S3,S4,S5 / S5,S6 |
| `w2q28` / `q88` | Past year: how many times used BNPL / how many late fees? | Frequency bands | S3,S4,S5 / S3,S5,S6 |
| `w2q29` | Unpaid CC balance after last payment (W2) | 0=No, 1=Yes | S4 |
| `w2q30` | Has credit card autopay (W2) | 0/1 | S4 |
| `w2q31` | Credit card autopay amount (W2) | 1=Minimum, 2=Full, 3=Other | S4,S5 |
| `q80` | Past year: unexpected CC cancellation or limit reduction | 0=No, 1=Yes | S3,S5,S6 |
| `[S4,S5,S6]q87` | How do you think credit score has changed? | 1=Improved, 2=Stayed same, 3=Gotten worse | S4,S5,S6 |

---

## 10. Credit Applications & Access (30 vars)

### Core Credit Application (all samples)
| Column (S3,S4) | Column (S5,S6) | Description | Values |
|----------------|----------------|-------------|--------|
| `q63` | `q76` | Past year: Applied for any type of credit or loan? | 0=No, 1=Yes |
| `q64` | `q77` | Past year: Turned down / not given as much credit? | 0/1 |
| `q65` | `q78` | Past year: Did not apply because thought would be turned down? | 0/1 |

### S6 Only
| Column | Description |
|--------|------------|
| `q66a–q66h` | Factors considered when applying: fees, interest rates, rewards, monthly payment, likelihood of qualifying, reputation, recommendation, other |
| `q67` | Thought of applying but anticipated being turned down |
| `q68a–q68f` | Changed mind because: interest too high, couldn't afford, didn't want debt, didn't trust lenders, didn't understand, other |
| `q69` | Past year: Refinanced any loans? |

### Wave 2
| Column | Description | Samples |
|--------|------------|---------|
| `w2q15` | Past year: Applied for credit or loan? | S3,S4,S5 |
| `w2q16` | Turned down for loan? | S3,S4,S5 |
| `w2q17` | Did not apply because thought would be turned down? | S3,S4,S5 |

---

## 11. Auto & Vehicle Loans (5 vars)

| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q97` / `q78` | Do you have an auto loan? | 0=No, 1=Yes | S3,S5,S6 / S4,S5,S6 |
| `q98` / `q79` | Did you get auto loan through dealer or by going to lender yourself? | 1=Through dealer, 2=By going to lender myself, 3=Don't know | S3,S5,S6 / S4,S5,S6 |
| `q99` | Past 2 years: paid off/prepaid an auto loan? | 1=No, 2=Yes turned in car, 3=Yes paid off early, 4=Yes paid on time | S3,S4,S6 |

---

## 12. BNPL (Buy Now Pay Later) — Key Variables (8 vars)

### Column Mapping
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q84` (S3) / `q77` (S4,S5,S6) | Past year: purchased something using BNPL? | 0=No, 1=Yes | All |
| `q85` | Past year: how many times used BNPL? | 1=1-2 times, 2=3-6 times, 3=More than 6 times | S3 |
| `q86` | Total amount of merchandise bought with BNPL | Numeric (topcoded) | S3 |
| `[S3]q87` | Paid interest or late fees for BNPL in past year? | 0=No, 1=Yes | S3 |
| `q88` | Last time incurred BNPL late fee: was it expected? | 0=No/unexpected, 1=Yes/expected | S3 |
| `q89` | Past 12 months: HH overdrafted or payment turned down? | 1=Yes overdraft, 2=Yes payment turned down | S3 |
| `w2q27` | Past 12 months: incurred late fee on any credit cards? | 0=No, 1=Yes | S3,S4,S5 |
| `w2q28` | Past year: how many times used BNPL (W2) | 1=Never, 2=1-2 times, 3=3-6 times, 4=More than 6 times | S3,S4,S5 |

### Harmonized BNPL Frequency (4 waves, identical wording)
| Wave | PDF Q# | CSV Column | Wording |
|------|--------|------------|---------|
| S4W2 | Q25 | `w2q25` | "...paid in four or fewer interest-free installments" |
| S5W1 | Q69 | `q69` | "...paid in four or fewer interest-free installments" |
| S5W2 | Q37 | `w2q55` | "...paid in four or fewer interest-free installments" |
| S6W1 | Q83 | `q77` | "...paid in four or fewer interest-free installments" |

All use 4-point scale: 1=Never, 2=1-2 times, 3=3-6 times, 4=More than 6 times.

### BNPL Missed Payment (2 waves)
| Wave | PDF Q# | CSV Column |
|------|--------|------------|
| S5W2 | Q48 | `w2q30h` |
| S6W1 | Q51 | `q45h` |

---

## 13. Alternative Financial Services (20 vars)

Payday loans, pawn shops, auto title loans, and Earned Wage Access (EWA).

| Column | Description | Samples |
|--------|------------|---------|
| `q79` / `w2q22` | Past 12 months: taken out payday loan? | All |
| `q80a–q80e` | What used payday loan for: regular HH expenses, purchase/repairs, healthcare, entertainment, other | S3 |
| `q81` | How many times rolled over payday loan? | S3 |
| `q82` | For last payday loan: repaid, rolled over, payment plan, stopped paying? | S3 |
| `q83` | How much currently owe on all payday loans? | S3 |
| `q92` / `w2q23` | Taken out pawn shop loan? | S3,S4,S5,S6 |
| `q93` / `w2q24` | Taken out auto title loan? | S3,S4,S5 |
| `q70` | Past 12 months: took out pawn shop loan | S5,S6 |
| `q71` | Past 12 months: took out auto title loan | S5,S6 |
| `q72` | Past 12 months: used Earned Wage Access (EWA) | 1=No, 2=Yes with app connected to employer, 3=Yes with app not connected | S5,S6 |
| `q73` | Paid fee for earned wage access? | 0=No, 1=Yes | S5,S6 |
| `q74` | Paid fee/tip to access wages quickly? | 0=No, 1=Yes | S5,S6 |
| `q75` | Past 12 months: payday loan or continued to owe? | 0=No, 1=Yes | S5,S6 |
| `q76` | Past 12 months: rolled over payday loan? | 0/1 | S5,S6 |

---

## 14. Bank Account & Overdraft/NSF (12 vars)

| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `[S3]q87` / `q89` | Past 12 months: HH overdrafted/payment turned down? | 1=Yes overdraft, 2=Yes payment turned down | S3 / S4,S5,S6 |
| `q88` / `q90` | How often did HH overdraft/have payment turned down? | 1=1-3, 2=4-10, 3=10+ | S3,S4,S5,S6 |
| `q91` | Last time overdrafted: surprised? | 1=Surprised, 2=Thought possible, 3=Expected | S3,S4,S5,S6 |
| `w2q18` | Past year: number of overdraft fees charged | 1=None, 2=1-3, 3=4-10, 4=10+ | S3,S4,S5 |
| `w2q19` | Last time: surprised by overdraft? | 1=Surprised, 2=Thought possible, 3=Expected | S3,S4,S5 |
| `w2q20` | Past year: number of NSF fees | 1=None, 2=1-3, 3=3+ | S3,S4,S5 |
| `w2q21` | Last time NSF: surprised or expected? | 1=Surprised, 2=Thought possible, 3=Expected | S3,S4,S5 |
| `[S5,S6]q87` | Past 12 months: overdraft fees | 1=None, 2=1-3, 3=4-10, 4=10+ | S5,S6 |
| `q88` (S5,S6) | Past 12 months: NSF fees | 1=None, 2=1-3, 3=3+ | S5,S6 |

---

## 15. Debt Collection (40 vars)

**S3 only** — detailed battery on debt collector interactions.

| Column | Description |
|--------|------------|
| `q59` | Past year (2021): contacted by debt collector? |
| `q60` | Number of separate debts in collection |
| `q61` | Type of debt most recently contacted about (12 types: credit card, mortgage, auto, student, medical, payday, taxes, telecom, utility, rent, other, don't know) |
| `q62` | How recently contacted? |
| `q63` | When first contacted? |
| `q64a–q64h` | Did collector: provide accurate info, contact too often, call before 8am/after 9pm, respect wishes, harass, state reason, treat politely, threaten |
| `q65` | How often did collector call? |
| `q66` | How often did you speak by phone? |
| `q67` | Collector attempt contact by email? |
| `q68` | Collector attempt contact by text? |
| `q69` | At first contact, believed you owed full amount? |
| `q70` | Paid off some/all of this debt? |
| `q71` | Received a notice from collector? |
| `q72a–q72d` | How useful was the notice? |
| `q73` | Disputed or attempted to verify debt? |
| `q74a–q74g` | What did you do: called collector, mailed form, mailed letter, contacted other way, contacted original creditor, contacted credit bureau, contacted lawyer |
| `q75a–q75d` | After dispute: debt collector verified, removed from credit report, offered settlement, stopped contacting |

---

## 16. Financial Difficulty & Bill Payment (100+ vars)

### Had Difficulty Paying
| Column | Description | Samples |
|--------|------------|---------|
| `q14` | Past 12 months: had difficulty paying for a bill or expense? 0=No, 1=Yes | S3,S4 |
| `w2q12` | Same question (W2) | S3,S4,S5 |
| `q40` | Same question | S5,S6 |

### Types of Expenses Causing Difficulty
**0**=No, **1**=Yes. Coverage varies:

| Category | S3 | S4 | S5 | S6 |
|----------|----|----|----|----|
| Medical expense | — | q41a | q44a | q44a |
| Car/vehicle repair | — | q41b | q44b | q44b |
| Home repair | — | q41c | q44c | q44c |
| Food | — | q41d | q44d | q44d |
| Mortgage/rent | — | q41e | q44e | q44e |
| Utilities | — | q41f | q44f | q44f |
| Taxes/fees | — | q41g | q44g | q44g |
| Death/funeral | — | q41h | q44h | q44h |
| Student loan/tuition | — | q41i | q44i | q44i |
| Childcare | — | q41j | q44j | q44j |
| Other HH expenses | — | q41k | q44k | q44k |
| Other | — | q41l | q44l | q44l |

### Actions Taken to Address Difficulty
Check-all-that-apply: **0**=Not Selected, **1**=Selected.

| Action | S3,S4 | S5,S6 |
|--------|-------|-------|
| Didn't pay all / paid late | q42a | q45a |
| Negotiated lower payment | q42b | q45b |
| Used non-retirement savings | q42c | q45c |
| Used retirement savings | — | q45d |
| Sold/pawned something | — | q45e |
| Cut expenses | — | q45f |
| Paid late/skipped other payments | — | q45g |
| Increased income (2nd job, OT) | — | q45h |
| Donated plasma | — | q45i |
| Used CC paid over time | — | q45j |
| Borrowed from friends/family | — | q45k |
| Used HELOC | — | q45l |
| Took out loan from institution | — | q45m |
| Took out payday/auto title loan | — | q45n |
| Cash advance from employer | — | q45o |
| Other | — | q45p |

### Frequency of Trouble
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `w2q13` | How often had trouble in last 12 months? | 1=Only once → 5=More than 12 times | S3,S4,S5 |
| `q42` (S5,S6) | Same question | Same | S5,S6 |

### Food Sufficiency
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q46` | Food sufficiency | 1=Enough of wanted food, 2=Enough not always wanted, 3=Sometimes not enough, 4=Often not enough | All |

---

## 17. Unexpected Expenses & Income Shocks (245+ vars)

This is the largest category. Each sample has batteries of unexpected expense and income drop questions with two sub-variables each: `_1` = indicator (0/1 did it happen), `_2` = amount (continuous, topcoded).

### Unexpected Expenses
Types covered: major medical/dental, unplanned gift/loan to family, major vehicle repair, major house/appliance repair, computer/phone repair, legal/taxes/fines, childcare increase, moving costs, other.

| Sample | Indicator + Amount Columns |
|--------|---------------------------|
| S3 W1 | `q27a1–q27h2` |
| S3 W2 | `w2q30a1–w2q30i2` |
| S4 W1 | `q33a1–q33i2` |
| S4 W2 | `w2q38a1–w2q38i2` |
| S5 W1 | `q37a1–q37i2` |
| S5 W2 | `w2q20a1–w2q20i2` |
| S6 W1 | `q34a1–q34i2` |

### Income Drops
Types covered: unemployment/furlough, reduced hours, reduced wages, lower-paying job, lost benefits, worked less due to illness/injury, worked less to care for sick, worked less for childcare, lost rental income, lost business revenue, other.

| Sample | Indicator + Amount Columns |
|--------|---------------------------|
| S3 W2 | `w2q31a1–w2q31l2` |
| S4 W1 | `q34a1–q34l2` |
| S4 W2 | `w2q39a1–w2q39l2` |
| S5 W1 | `q38a1–q38l2` |
| S5 W2 | `w2q21a1–w2q21k2` |
| S6 W1 | `q35a1–q35k2` |

---

## 18. Life Events & Household Changes (52 vars)

Check-all-that-apply batteries: "In the past 12 months, has any of the following happened?" **0**=No, **1**=Yes.

| Event | S3W2 | S4W1 | S4W2 | S5W1 | S5W2 | S6W1 |
|-------|------|------|------|------|------|------|
| Someone in HH got married | — | — | — | — | w2q22a | — |
| Someone divorced/separated | w2q29b | q32b | w2q37b | q39b | w2q22b | q36b |
| Someone had major illness/injury | w2q29c | q32c | w2q37c | q39c | w2q22c | q36c |
| Someone died | — | — | — | — | w2q22d | — |
| Someone born/adopted/moved in | w2q29e | q32e | w2q37e | q39e | w2q22e | q36e |
| Someone left HH | w2q29f | q32f | w2q37f | q39f | w2q22f | q36f |
| Someone in HH retired | w2q29g | q32g | w2q37g | q39g | w2q22g | q36g |
| Car was repossessed | — | — | — | — | w2q22h | — |
| Someone arrested/charged/incarcerated | — | — | — | — | w2q22i | — |
| You moved to new residence | w2q29j | q32j | w2q37j | q39j | w2q22j | q36j |
| You started a new business | w2q29l | q32l | w2q37l | q39l | w2q22l | q36l |
| You closed a business | w2q29m | q32m | w2q37m | q39m | w2q22m | q36m |

---

## 19. Fraud, Scams & Identity Theft (71 vars)

### S6 Fraud (q98a–q103g2)
| Column | Description |
|--------|------------|
| `q98a–q98k` | Past year: lost money to (11 types): theft/misuse by known person, credit/debit card fraud, imposter scams, phishing, investment scams, prize/lottery scams, fake job opportunities, online shopping scams, identity theft, social media scams, other |
| `q99` | Past year: number of times lost money to scam/fraud |
| `q100` | Ultimately lost money in most recent scam? (5 categories: all recovered → none) |
| `q101` | Amount lost in most recent scam (continuous) |
| `q102a–q102j` | Product involved in most recent scam: credit card, debit card, P2P apps, wire transfer, personal check, cashier's check, cash withdrawal, loan, virtual currencies, other |
| `q103a1–q103g2` | Reported to + helped resolve: bank/FI, credit card company, credit bureaus, local law enforcement, federal agency, BBB, other |

### S5 W2 Fraud (`w2q66–w2q70`)
Same structure as S6.

---

## 20. Credit Repair & Monitoring (9 vars)

| Column | Description | Samples |
|--------|------------|---------|
| `q89` | Past year: paid company/service to repair/improve credit | S5,S6 |
| `q90` | Was there a specific reason to improve credit? | S5,S6 |
| `q91a–q91h` | Reason: qualify for mortgage, qualify for other credit, rent apartment, get hired, refinance, business credit, general health, other | S6 |
| `q92a–q92e` | Type of service paid: credit repair, credit counseling, debt consolidation, debt relief/settlement, other | S6 |
| `q93` | Amount spent on credit improvement (bands) | S6 |
| `q94` | Required to pay before results? 0=No, 1=Yes | S6 |
| `q95` | Success of credit improvement service (1=Very successful → 5=I don't know) | S6 |
| `q96` | Past year: disputed item on credit report? 0=No, 1=Yes | S6 |
| `q97` | Dispute resulted in removal? 0/1 | S6 |
| `w2q64` | Amount spent on credit improvement | S4,S5 |
| `w2q65` | Required to pay before results? | S4,S5 |

### Credit Score Check
| Column | Description | Values | Samples |
|--------|------------|--------|---------|
| `q60` | When last checked credit score/report | 1=Never, 2=At least one year ago, 3=Within the year | All |

---

## 21. Financial Information Sources (17 vars)

**S6 (q61a–q61h):** Sources used for financial info: family/friends, financial planner/adviser/broker, internet/web service/app, banker/lawyer/accountant, books/magazines/newspapers, podcasts/radio/TV, conferences/workshops, social media.

**S6 social media detail (q62a–q62j):** TikTok, Twitter/X, Facebook, WhatsApp, YouTube, Instagram, Snapchat, Reddit, Discord, Other.

**S5 W2 (w2q43a–w2q43h + w2q44i–w2q44j):** Same structure.

---

## 22. Sports Betting & Gambling (6 vars)

**S6 only.**

| Column | Description | Values |
|--------|------------|--------|
| `q117` | Past year: placed a sports bet? | 0=No, 1=Yes |
| `q118` | Used mobile phone for sports bet? | 0/1 |
| `q119` | Frequency of sports betting | 1=Once → 6=Daily |
| `q120` | Typical bet amount | 1=$20 or less → 5=More than $250 |
| `q121` | Largest single-day loss | 1=$50 or less → 5=More than $500 |
| `q122` | Past year: gambled in casino? | 0=No, 1=Yes |

---

## 23. Time Preferences & Risk Expectations (8 vars)

### Time Preference (S6 only)
Respondents choose between $1000 in 1 month vs. a larger amount in 6 months.

| Column | Choice |
|--------|--------|
| `q123` | $1000 in 1 month vs $1050 in 6 months |
| `q124` | $1000 in 1 month vs $1100 in 6 months |
| `q125` | $1000 in 1 month vs $1150 in 6 months |

### Probability Expectations (S6 only, 0–100 scale)
| Column | Description |
|--------|------------|
| `q126a` | % chance of moving to new residence next year |
| `q126b` | % chance of 20-point credit score decrease |
| `q126c` | % chance of income drop due to unemployment/furlough |
| `q126d` | % chance of major out-of-pocket medical expense |
| `q126e` | % chance of major vehicle repair/replacement |
| `q126f` | % chance of major house/appliance repair |

---

## Standard Value Labels Reference

Most variables use these standard codings:

| Pattern | Coding |
|---------|--------|
| Binary (Yes/No) | 0=No, 1=Yes |
| Check-all-that-apply | 0=Not Selected, 1=Selected |
| Financial wellbeing (q1a–q1e) | 1=Completely, 2=Very Well, 3=Somewhat, 4=Very Little, 5=Not at all |
| Financial behavior (q2a–q2b) | 1=Always, 2=Often, 3=Sometimes, 4=Rarely, 5=Never |
| Frequency | 1=Never/None, then increasing bands |
| Income bands | 1=lowest → 10/11=highest |
| Missing | `.` (dot) in CSV |

---

## Feature Availability Matrix (for Harmonized Modeling)

| Feature Group | S4W2 | S5W1 | S5W2 | S6W1 |
|--------------|:----:|:----:|:----:|:----:|
| Demographics (age, sex, race, education, partner, housing, military) | YES | YES | YES | YES |
| Income (derived or raw q10) | derived | derived | derived | raw q10 |
| Financial Well-Being (fwb) | YES | YES | YES | YES |
| HH Composition (q106, q107) | YES | YES | YES | YES |
| Financial Stress (q12/q45) | q12 | q12 | q12 | q45 |
| Work Status (q3a/b) | YES | YES | YES | YES |
| BNPL Frequency | w2q25 | q69 | w2q55 | q77 |
| BNPL Missed Payment | — | — | w2q30h | q45h |
| Risk Preference | — | — | — | q123–q125 |
| Financial Literacy | — | — | — | q1a–q1e |
| Sports Betting | — | — | — | q117–q122 |
| Fraud/Scams | — | — | w2q66–w2q70 | q98a–q103g2 |
| Unexpected Expenses | w2q38a1–i2 | q37a1–i2 | w2q20a1–i2 | q34a1–i2 |
| Income Shocks | w2q39a1–l2 | q38a1–l2 | w2q21a1–k2 | q35a1–k2 |

---

## Key Cross-Sample Question Mappings

| Construct | S3 | S4 | S5 | S6 |
|-----------|----|----|----|-----|
| Financial wellbeing (W1) | q1a–q1d,q2a–q2b | q1a–q1e,q2a–q2b | q1a–q1e,q2a–q2b | q1a–q1e,q2a–q2b |
| Work status | q3a1–q3b7 | q3a1–q3b7 | q3a1–q3b7 | q3a1–q3b7 |
| Had difficulty paying | q14 | q14 | q40 | q40 |
| Difficulty expense types | (none in W1) | q41a–q41o | q44a–q44l | q44a–q44l |
| Actions after difficulty | q42a–q42c | q42a–q42c | q45a–q45o | q45a–q45o |
| Unexpected expenses | q27a1–q27h2 | q33a1–q33i2 | q37a1–q37i2 | q34a1–q34i2 |
| Income drops | (none in W1) | q34a1–q34l2 | q38a1–q38l2 | q35a1–q35k2 |
| Payday loan | q79 | q79 | q75 | q75 |
| Auto loan | q97 | (q78) | q78 | q78 |
| Credit card | q95 | q95 | q81 | q81 |
| Medical debt | q50–q58 | q50–q58 | q50–q58 | q50–q58 |
| BNPL usage | q84–q89 | q84 | q77 | q77 |
| Credit applied/rejected | q76–q78 | q76–q78 | q63–q65 | q63–q69 |
| Marital status | q110a–q110f | q105a–q105f | q105a–q105f | q105a–q105f |
| Other adults in HH | q102 | q106 | q102 | q106 |
| Children in HH | q103 | q107 | q103 | q107 |
