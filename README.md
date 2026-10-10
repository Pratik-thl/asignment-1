# MapleFreight Late-Delivery Prediction with Slack Alerts

AML 3303 Assessment 1 · Student ID c0968446

## Overview

MapleFreight handles roughly 8,000 shipments a week across Ontario, Quebec and the Prairies. Its on-time rate has dropped from 94% to 88%, and late loads are often noticed only after the promised delivery window has passed.

This project predicts which shipments are at risk of arriving late and posts the high-risk ones to Slack (`#dispatch-alerts`) early enough for dispatch to act.

## Dataset

`data/maplefreight_delivery_delay_dataset.csv` contains 6,035 shipments and 23 columns.

| Item | Detail |
|------|--------|
| Target | `delivered_late` |
| Late share | ~21% |
| Leakage column dropped | `actual_transit_hours` |
| Cleaning | Duplicates removed; `service_level` casing standardised; missing values imputed inside the pipeline |

## Project structure

```
.
├── data/maplefreight_delivery_delay_dataset.csv
├── screenshots/slack_alert.png
├── MapleFreight_Delivery_Delay_Prediction_c0968446.ipynb
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

## Getting started

1. Create and activate a virtual environment, then install the dependencies:

```bash
   python -m venv .venv
   # Windows
   .venv\Scripts\activate
   # macOS / Linux
   source .venv/bin/activate
   pip install -r requirements.txt
```

2. In Slack, create the `#dispatch-alerts` channel and add an Incoming Webhook.
3. Copy `.env.example` to `.env` and set `SLACK_WEBHOOK_URL`.
4. Open the notebook and run all cells from a clean kernel.

## Modelling

| Item | Choice |
|------|--------|
| Baseline | Majority-class dummy classifier |
| Models compared | Balanced logistic regression, random forest, AdaBoost |
| Final model | Balanced logistic regression (highest late-class F1) |
| Train/test split | 78/22 stratified, `random_state=33` |
| Engineered features | `fuel_per_km`, `is_express`, `long_haul`, `busy_route`, province codes |
| Alert threshold | 0.52 |
| Holdout results at 0.52 | Late class: precision ~0.43, recall ~0.71, F1 ~0.54 |

**Why 0.52?** Of the thresholds that kept late-class recall at 0.70 or above, 0.52 gave the best F1 and sent fewer alerts than a lower cut would.

## Key findings

- Contract Owner-Op shipments are late more often than MapleFreight Fleet shipments.
- Storm weather produces clearly more late deliveries than clear days.
- Long-haul and busy-route shipments have higher late rates.
- Express service is slightly more reliable than Standard or Economy in this dataset.

## Recommendations

1. Hold Economy freight on long-haul routes during storm days.
2. Limit the owner-operator share on premium delivery windows until their late counts improve.
3. Recalculate the ETA whenever pickup is already running late.
4. Prioritise fleet capacity on busy long-haul lanes.
5. Pilot the Slack alerts at 0.52, then raise the threshold if dispatch wants fewer alerts.

## Submission checklist

- [x] Notebook named with student ID and run with outputs
- [x] Baseline and final model with precision, recall, F1 and confusion matrix
- [x] Threshold chosen and justified
- [x] `screenshots/slack_alert.png`
- [x] `.env` gitignored and `.env.example` present
- [x] Dataset included
