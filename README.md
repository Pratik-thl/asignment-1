# Predicting late MapleFreight deliveries (Slack to dispatch)

AML 3303 Assessment 1 · Student ID c0968446

## Problem

MapleFreight moves about 8,000 shipments a week across Ontario, Quebec and the Prairies. On-time performance fell from 94% to 88%. Late loads often surface only after the promised window.

This project scores late risk and posts high-risk shipments to Slack (`#dispatch-alerts`) while there is still time to react.

## Dataset

`data/maplefreight_delivery_delay_dataset.csv` - 6,035 shipments, 23 columns.

| Item | Detail |
|------|--------|
| Target | `delivered_late` |
| Late share | ~21% |
| Leakage dropped | `actual_transit_hours` |
| Cleaning | duplicates removed; `service_level` casing fixed; missing values imputed in-pipeline |

## Layout

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

## How to run

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate
pip install -r requirements.txt
```

1. Create `#dispatch-alerts` and an Incoming Webhook  
2. Copy `.env.example` to `.env` and set `SLACK_WEBHOOK_URL`  
3. Open the notebook and run all cells from a clean kernel

## Modelling

| Item | Choice |
|------|--------|
| Baseline | Majority-class dummy |
| Compared | Balanced logistic regression, random forest, AdaBoost |
| Selected | Balanced logistic regression (best late-class F1) |
| Split | 78/22 stratified, `random_state=33` |
| Extra features | `fuel_per_km`, `is_express`, `long_haul`, `busy_route`, province codes |
| Alert threshold | 0.52 |
| Holdout @ 0.52 | Precision ~0.43, recall ~0.71, F1 ~0.54 on the late class |

Why 0.52: among cuts with late recall ≥ 0.70, 0.52 had the strongest F1 without flooding the alert queue as much as a lower cut.

## Findings

- Contract Owner-Op is late more often than MapleFreight Fleet.
- Storm weather is clearly worse than clear days.
- Long-haul and busy-route flags line up with higher late rates.
- Express service is a bit safer than Standard/Economy in this export.

## Recommendations

1. Hold Economy freight on storm days for long hauls.  
2. Limit owner-op share on premium windows until recent late counts improve.  
3. Re-check ETA when pickup is already drifting late.  
4. Prefer fleet capacity on busy long-haul lanes.  
5. Pilot Slack at 0.52, then raise it if dispatch wants fewer alerts.



## Checklist

- [x] Notebook named with student ID and run with outputs  
- [x] Baseline + final model with precision / recall / F1 / confusion matrix  
- [x] Threshold chosen and justified  
- [x] `screenshots/slack_alert.png`  
- [x] `.env` gitignored + `.env.example` present  
- [x] Dataset included  
