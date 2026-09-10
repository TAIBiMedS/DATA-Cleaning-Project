
# Marketing Campaign Data Cleaning

A Python/Pandas project that takes a messy, real-world-style marketing campaign
dataset and turns it into an analysis-ready dataset — fixing inconsistent
formatting, typos, bad types, logical errors, and outliers along the way.

## Overview

Raw marketing data is rarely clean: currency symbols stuck to numbers,
the same category spelled five different ways, dates in mixed formats,
booleans stored as `Y`/`No`/`1`/`True`, and rows where a campaign somehow
ends before it starts. This project walks through cleaning one such dataset
(2,020 rows of ad campaign records) step by step, with printed before/after
checks at every stage so each transformation is easy to verify.

## Dataset

| File | Description |
|---|---|
| `data/marketing_campaign_data_messy.csv` | Raw input data, as originally exported |
| `data/cleaned_marketing_campaign_data.csv` | Final cleaned output |

**Columns:** `campaign_id`, `campaign_name`, `start_date`, `end_date`, `channel`,
`impressions`, `clicks`, `spend`, `conversions`, `active`, `campaign_tag`, `season`
(the last is engineered during cleaning — see below).

## What was wrong with the data, and how it was fixed

| Problem | Fix |
|---|---|
| Inconsistent column names (spacing, casing) | Stripped, lowercased, and underscored all column headers |
| Duplicate `Clicks` column (one empty) | Dropped the duplicate column |
| `spend` stored as text with `$` and commas | Stripped non-numeric characters with regex, cast to `float` |
| `channel` had typos/variants (`Facebok`, `Insta_gram`, `E-mail`, `Tik_Tok`, `Gogle`, `N/A`) | Mapped to standardized labels (`Facebook`, `Instagram`, `Email`, `TikTok`, `Google Ads`, `NaN`) |
| `active` stored inconsistently (`Y`, `No`, `Yes`, `True`, `'0'`, `'1'`) | Mapped everything to a clean `0`/`1` integer flag |
| `start_date` mixing `YYYY-MM-DD 00:00:00` and plain dates | Stripped trailing timestamps, parsed with `pd.to_datetime(..., format="mixed", dayfirst=True)` |
| `end_date` in a fixed format with some bad values | Parsed with `pd.to_datetime(..., errors="coerce")` |
| Campaigns where `start_date` is after `end_date` ("time travel" rows) | Reset `end_date` to `start_date + 30 days`, matching the typical campaign length in this dataset |
| Extreme outliers in `spend` | Capped values above `Q3 + 3×IQR` at that upper bound (Tukey's rule, widened for a lenient cap) |
| No `season` field, but it's embedded in the campaign name | Extracted with a regex from `campaign_name` (e.g. `Q4_Summer_CMP-00001` → `Summer`) |

A quick sanity check (`clicks > impressions`) was also run to catch physically
impossible rows, since a click can't be logged without an impression.

## Repo structure

```
.
├── data/
│   ├── marketing_campaign_data_messy.csv     # raw input
│   └── cleaned_marketing_campaign_data.csv   # cleaned output
├── notebooks/
│   └── marketing_data_cleaning.ipynb         # full step-by-step cleaning notebook
├── requirements.txt
├── LICENSE
└── README.md
```

## Tech stack

- Python 3
- pandas, numpy — cleaning and transformation
- matplotlib, seaborn — distribution and spend-by-channel visualizations

## Running it

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
pip install -r requirements.txt
jupyter notebook notebooks/marketing_data_cleaning.ipynb
```

The notebook reads `data/marketing_campaign_data_messy.csv` — update the file
path at the top of the notebook if you move the data folder.

## Notes / things to know

- This is a cleaning-focused project, not a full EDA — the notebook includes
  two exploratory charts (spend distribution, spend by channel) mainly to
  sanity-check the cleaned numbers, not as a complete analysis.
- The `channel` typo-mapping and the "30-day default campaign length" rule
  are dataset-specific judgment calls, documented inline in the notebook —
  worth calling out if someone reuses this on different data.



