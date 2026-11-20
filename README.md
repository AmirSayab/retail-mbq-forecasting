# Retail MBQ Forecasting

> A full-stack planning system that turns spreadsheet inventory into explainable MBQ rules, demand forecasts, and operational dashboards.

![Python](https://img.shields.io/badge/Python-3.10+-blue) ![LightGBM](https://img.shields.io/badge/LightGBM-demand_forecast-orange) ![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green) ![License](https://img.shields.io/badge/License-Private-gray)

## Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Excel upload │────▶│ Harmonize    │────▶│ SQLAlchemy   │
│ (81k+ rows)  │     │ (clean/norm) │     │ (SQLite)     │
└──────────────┘     └──────────────┘     └──────┬───────┘
                                                  │
                     ┌────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                    MBQ RULE ENGINE                            │
│                                                              │
│  1. Select reference periods (same month LY + 2 prior)      │
│  2. Calculate CBS / SLS / STR per period                     │
│  3. Apply sell-through thresholds (≤2 → 6× SLS rule)        │
│  4. Apply store-category min/max clamp                       │
│  5. Apply transit adjustment factor                          │
│  6. Allocate: Color → Price Band → Size                     │
│                                                              │
└────────────────────────────┬────────────────────────────────┘
                             │ baseline features
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    ML FORECASTING                            │
│  Feature engine (30/60/90-day) │ LightGBM demand forecast   │
│  Dynamic MBQ adjustment        │ ML-enhanced allocation     │
└────────────────────────────┬────────────────────────────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
     ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
     │ Sales trend  │ │ Dead stock   │ │ Validation   │
     │ API          │ │ detect       │ │ harness      │
     │              │ │ (30-45+ days)│ │ (vs Excel)   │
     └──────────────┘ └──────────────┘ └──────────────┘
              │
              ▼
     ┌─────────────────────────────────────────────────┐
     │            REACT DASHBOARD                       │
     │  Upload │ MBQ Planner │ Forecasting │ Dead Stock │
     │  (Recharts visualizations)                       │
     └─────────────────────────────────────────────────┘
```

## Problem

Retail planners were using large spreadsheets to calculate Minimum Base Quantity (MBQ) across color, price band, and size dimensions. The workflow mixed historical sell-through, store rules, transit factors, and manual checks — making changes difficult to audit.

## Approach

**Rule engine preserves explainability — ML is an additive signal.**

The FastAPI backend reproduces the exact business calculation sequence:
1. Select reference periods (same month last year + 2 prior)
2. Calculate closing stock (CBS), sales (SLS), and sell-through rate (STR)
3. Apply sell-through thresholds and store-specific bounds
4. Apply transit adjustment
5. Allocate the color-level result into price band and size contributions

LightGBM modules add 30/60/90-day demand forecasts and an optional learned MBQ adjustment — but the rule engine remains the auditable baseline.

## Evidence

| Metric | Value | Context |
|--------|-------|---------|
| Ingestion scale | **81,000+ rows** | Documented in project specs |
| Allocation levels | **3** | Color → price band → size |
| Forecast horizons | **3** | 30, 60, and 90 days |

> The repository does not contain a committed forecast accuracy score. Architecture and documented scale are reported, not model lift.

## Tech Stack

- **Backend**: FastAPI, SQLAlchemy (SQLite), Python
- **ML**: LightGBM (demand forecasting)
- **Frontend**: React, Recharts
- **Analytics**: Dead stock detection (30-45+ days), sales trend API
- **Validation**: Harness comparing computed MBQ vs original Excel workbook

## Key Design Decisions

1. **Preserve the rule baseline** — planners can reconcile against the original workbook
2. **ML as additional signal** — doesn't replace business controls with unexplained numbers
3. **Three allocation levels** — color, price band, size maintain inventory granularity
4. **Validation harness** — compares computed output against reference Excel sheets

## What I Would Improve

- Version input schemas and commit de-identified evaluation fixtures
- Report reconciliation rate and forecast MAE by category
- Add drift monitoring for seasonal pattern changes
- Run shadow forecasts before any automated purchasing decision

## Disclosure

Customer inventory and store/SKU data remain private. Dataset scale comes from project documentation; no forecast accuracy is claimed. Built as a client project.

---

**Portfolio**: [amir-sayab-portfolio.vercel.app](https://amir-sayab-portfolio.vercel.app/work/retail-mbq-forecasting/) · **LinkedIn**: [linkedin.com/in/amir-sayab](https://www.linkedin.com/in/amir-sayab/)
