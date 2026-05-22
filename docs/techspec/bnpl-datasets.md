# BNPL (Buy Now Pay Later) — Dataset Mapping

## Overview

Mapping of CFPB "Making Ends Meet" survey sample PDFs in `docs/samples/` that contain Buy Now Pay Later (BNPL) questions. BNPL questions were first introduced in Sample 3 Wave 1 (January 2022) and have been carried forward through subsequent waves with refinements.

## Datasets WITHOUT BNPL

The following files predate BNPL-specific questions and contain no mentions:

- `CFPB MEM Sample 1 Wave 1 (May 2019).pdf`
- `CFPB MEM Sample 1 Wave 2 (May 2020).pdf`
- `CFPB MEM Sample 1 Wave 3 (Feb 2021).pdf`

## Datasets WITH BNPL

---

### 1. CFPB MEM Sample 3 Wave 1 (Jan 2022)

First wave to introduce BNPL-specific questions. These are simple yes/no questions (not yet multi-choice frequency).

**Q85.** In the past year, how many times have you purchased something using a "buy now, pay later" option?

- No
- Yes

**Q85 follow-up.** ...pay for the full price at the time of purchase, but rather paid in four interest-free installments? (Some retailers offer these payment plans through companies such as Affirm, Afterpay, and Klarna.)

- Yes
- No

**Q87.** Have you paid interest or late fees for a "buy now, pay later" purchase in the past year?

---

### 2. CFPB MEM Sample 3 Wave 2 (Jan 2023)

BNPL question upgraded to multi-choice frequency scale.

**Q28.** In the past year, how many times have you purchased something using a "buy now, pay later" option, in which you did not pay for the full price at the time of purchase, but rather paid in four interest-free installments? Some retailers offer these payment plans through companies such as Affirm, Afterpay, and Klarna.

- Never in the past year
- 1-2 times
- 3-6 times
- More than 6 times

---

### 3. CFPB MEM Sample 4 Wave 1 (Jan 2023)

Same question as Sample 3 Wave 2.

**Q71.** In the past year, how many times have you purchased something using a "buy now, pay later" option, in which you did not pay for the full price at the time of purchase, but rather paid in four interest-free installments? Some retailers offer these payment plans through companies such as Affirm, Afterpay, and Klarna.

- Never in the past year
- 1-2 times
- 3-6 times
- More than 6 times

---

### 4. CFPB MEM Sample 4 Wave 2 (Jan 2024)

Wording updated: "four or fewer interest-free installments" (added "or fewer").

**Q25.** In the past year, how many times have you purchased something using a "buy now, pay later" option, in which you did not pay for the full price at the time of purchase, but rather paid in four or fewer interest-free installments? Some retailers offer these payment plans through companies such as Affirm, Afterpay, and Klarna.

- Never in the past year
- 1-2 times
- 3-6 times
- More than 6 times

---

### 5. CFPB MEM Sample 5 Wave 1 (Jan 2024)

Same wording as Sample 4 Wave 2.

**Q69.** In the past year, how many times have you purchased something using a "buy now, pay later" option, in which you did not pay for the full price at the time of purchase, but rather paid in four or fewer interest-free installments? Some retailers offer these payment plans through companies such as Affirm, Afterpay, and Klarna.

- Never in the past year
- 1-2 times
- 3-6 times
- More than 6 times

---

### 6. CFPB MEM Sample 5 Wave 2 (Jan 2025)

First appearance of BNPL in a missed-payment checklist, alongside traditional credit products.

**Q37.** In the past year, how many times have you purchased something using a "buy now, pay later" option, in which you did not pay for the full price at the time of purchase, but rather paid in four or fewer interest-free installments? Some retailers offer these payment plans through companies such as Affirm, Afterpay, and Klarna.

- Never in the past year
- 1-2 times
- 3-6 times
- More than 6 times

**Q48.** Thinking back to the most recent time you had difficulty, did you miss a payment for any of the following? Please mark all that apply.

- Mortgage
- Credit card bill
- Student loan
- Auto loan
- Payday, auto title, or pawn loan
- Personal loan from a bank or other financial services provider
- Loan from a family member, friend, or acquaintance
- **Buy-now-pay-later payments**
- Cash advance from your employer
- Other (please specify):

---

### 7. CFPB MEM Sample 6 Wave 1 (Jan 2025)

Same structure as Sample 5 Wave 2 with different question numbers.

**Q83.** In the past year, how many times have you purchased something using a "buy now, pay later" option, in which you did not pay for the full price at the time of purchase, but rather paid in four or fewer interest-free installments? Some retailers offer these payment plans through companies such as Affirm, Afterpay, and Klarna.

- Never in the past year
- 1-2 times
- 3-6 times
- More than 6 times

**Q51.** Thinking back to the most recent time you had difficulty, did you miss a payment for any of the following? Please mark all that apply.

- Mortgage
- Credit card bill
- Student loan
- Auto loan
- Payday, auto title, or pawn loan
- Personal loan from a bank or other financial services provider
- Loan from a family member, friend, or acquaintance
- **Buy-now-pay-later payments**
- Cash advance from your employer
- Other (please specify):

---

## Summary of BNPL Question Evolution

| Wave | Date | Q# (Frequency) | Q# (Missed Payment) | Wording Change |
|---|---|---|---|---|
| S3W1 | Jan 2022 | Q85-87 | — | Yes/no usage + interest fees (first introduction) |
| S3W2 | Jan 2023 | Q28 | — | Upgraded to 4-point frequency scale |
| S4W1 | Jan 2023 | Q71 | — | Same wording |
| S4W2 | Jan 2024 | Q25 | — | "four or fewer" installments |
| S5W1 | Jan 2024 | Q69 | — | Same wording |
| S5W2 | Jan 2025 | Q37 | Q48 | Added missed-payment checklist with BNPL |
| S6W1 | Jan 2025 | Q83 | Q51 | Same structure |

## Data Merging Analysis

### Merging S5W2 and S6W1

Both surveys contain the same two BNPL variables with identical wording, making them the best candidates for merging.

**Same (identical wording):**
- BNPL frequency: Q37 (S5W2) ↔ Q83 (S6W1) — "four or fewer interest-free installments" with 4-point ordinal scale (Never / 1-2 / 3-6 / >6 times)
- Missed-payment BNPL checkbox: Q48 (S5W2) ↔ Q51 (S6W1) — same checklist with "Buy-now-pay-later payments" as a selectable option

**Different:**
- Survey length: S5W2 has ~60 questions, S6W1 has ~108 (longer form with extra sections on crypto, credit repair, insurance, small business, etc.)
- Question numbering does not align — same topics have different Q numbers between surveys
- Some covariate questions exist in only one survey

**Merge approach (row-binding):**
1. Map questions to a canonical variable schema by content (not by Q number)
2. Variables absent from one survey become `NaN` for those rows
3. Result: 2 survey waves with ~15–20 overlapping covariates + 2 BNPL variables

### Merging S4W2 + S5W1 + S5W2 + S6W1 (Frequency Only)

The BNPL frequency question is identical across all four waves (Q25 / Q69 / Q37 / Q83):
- Same wording: "four or fewer interest-free installments"
- Same 4-point ordinal scale
- Same cited companies (Affirm, Afterpay, Klarna)

This yields 4 waves across 2024–2025 for trend/panel analysis of BNPL adoption rates. However, only S5W2 and S6W1 include the missed-payment outcome variable.

## ML Recommendation

| Priority | Dataset(s) | Why |
|---|---|---|
| **Best** | S5W2 alone | BNPL frequency feature + missed-payment target + richest covariates + most recent (2025) |
| **Good** | S5W2 + S6W1 (merged) | Larger N, two waves, both variables available |
| **Trend** | S4W2 + S5W1 + S5W2 + S6W1 | 4 waves with identical frequency question for time-series |
| **Legacy** | S3W2 + S4W1 | Only frequency, older data (2023), binary-to-ordinal not aligned |

## Data Files & Structure

### Source

All data was downloaded from the CFPB Making Ends Meet Survey public data page:
[https://www.consumerfinance.gov/data-research/making-ends-meet-survey-data/public-data/](https://www.consumerfinance.gov/data-research/making-ends-meet-survey-data/public-data/)

Sample 2 was a small specialized survey and is omitted from public release.

### File organization

Each sample is distributed as a single ZIP archive at `docs/cfpb_making-ends-meet_data-sample-*.zip`. The PDFs in `docs/samples/` are **survey questionnaires only** — the actual respondent microdata lives inside these ZIPs.

### Per-sample contents

| Sample | ZIP File | CSV Data File | Stata File | Waves | Questionnaires (PDFs) |
|--------|----------|---------------|------------|-------|----------------------|
| S1 | `...sample-1.zip` | `MEM_S1W1W2W3_PUF.csv` | `.dta` | W1 (May 2019) + W2 (May 2020) + W3 (Feb 2021) | 3 PDFs |
| S3 | `...sample-3.zip` | `MEM_S3W1W2_PUF.csv` | `.dta` | W1 (Jan 2022) + W2 (Jan 2023) | 2 PDFs |
| S4 | `...sample-4.zip` | `MEM_S4W1W2_PUF.csv` | `.dta` | W1 (Jan 2023) + W2 (Jan 2024) | 2 PDFs |
| S5 | `...sample-5.zip` | `MEM_S5W1W2_PUF.csv` | `.dta` | W1 (Jan 2024) + W2 (Jan 2025) | 2 PDFs |
| S6 | `...sample-6.zip` | `MEM_S6W_PUF.csv` / `MEM_S6W1_PUF.csv` | `.dta` | W1 (Jan 2025) only | 1 PDF |

Each ZIP also contains a codebook (`*_codebook.txt`), a user guide (`MEM PUF User Guide.pdf`), and a `README.txt`.

### Column naming convention — wide format

Waves are merged into a single CSV in **wide format** (one row per respondent), using prefix-based column naming:

| Prefix | Wave | Example |
|--------|------|---------|
| (none) | Wave 1 | `q37`, `q48` |
| `w2` | Wave 2 | `w2q37`, `w2q48` |
| `w3` / `w321` | Wave 3 (S1 only) | `w3q*` / `w321*` |

This means a single row contains both W1 and W2 responses for the same respondent, differentiated by column prefix.

### Wave-specific variables

Each data file includes survey weight columns that vary by wave and mode:

| Suffix | Purpose |
|--------|---------|
| `weight` | Weights for Wave 1 cross-sectional analysis |
| `comb_weight` | Combined weight for cross-sample analysis |
| `w2weight` | Weights for panel analysis (W1→W2 attrition) |
| `w2weight_paper` / `w2weight_online` | Mode-specific W2 weights |
| `w2comb_weight_*` | Combined weight for W2 + other sample |

### BNPL question mapping (PDF Q# → CSV column)

| Sample | Wave | PDF Question | CSV Column |
|--------|------|-------------|------------|
| S3W1 | W1 | Q85, Q87 | `q85`, `q87` |
| S3W2 | W2 | Q28 | `w2q28` |
| S4W1 | W1 | Q71 | `q71` |
| S4W2 | W2 | Q25 | `w2q25` |
| S5W1 | W1 | Q69 | `q69` |
| S5W2 | W2 | Q37, Q48 | `w2q37`, `w2q48` |
| S6W1 | W1 | Q83, Q51 | `q83`, `q51` |

### ML implications for cross-sample merging

Since each sample's CSV uses wide format (columns per wave), merging across samples requires:

1. **Extract** the relevant wave columns from each sample's CSV
2. **Rename** columns to a common schema (e.g., strip `w2q` → `q` for Wave 2 columns)
3. **Row-bind** only if the renamed columns have conceptually identical questions
4. **Handle missing covariates** — non-overlapping questions become `NaN`

Because S5W2 columns are `w2q*` and S6W1 columns are plain `q*`, a direct merge of S5W2 + S6W1 would require dropping the `w2` prefix from S5W2 columns first, then binding rows. This only works for questions that exist in both surveys with identical wording.
