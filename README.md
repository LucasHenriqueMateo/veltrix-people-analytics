# Veltrix Group — People Analytics

**HR Workforce, Performance & Movement Analysis**

## Context

Veltrix Group is a fictional mid-size tech and services company (~650 employees). I built this dashboard to cover the full People Analytics cycle: current workforce snapshot, performance reviews, and the complete movement history (hires, terminations, promotions, transfers, raises) since the company's founding.

The goal was a single-page view that lets HR leadership track headcount health, turnover, and performance trends without pulling manual reports.

## Dataset

Three related tables, joined by `colaborador_id`:

- **rh_colaboradores.csv** (650 rows) — current employee snapshot: role, level, department, salary, contract type, status
- **rh_avaliacoes.csv** (~2,683 rows) — performance reviews from 2024-Q1 to 2025-Q2: technical, behavioral, and manager scores
- **rh_movimentacoes.csv** (~989 rows) — full movement history: hires, terminations, promotions, transfers, salary increases

The raw data had typical real-world issues: mixed date formats (ISO/BR), gender values written inconsistently, salaries using comma decimals, and duplicate movement records. These are documented in `power_query.md`.

## Dashboard

Published via Power BI Service. Single-page layout with:

- **5 KPI cards:** Active Headcount · Turnover % (12 months) · Average Salary · Average Performance Score · Promotions in Period
- **Headcount by department:** horizontal bar chart
- **Hires vs terminations trend:** monthly line chart, 2024–2025
- **Level distribution:** donut chart (Junior to Manager)
- **Review results by department:** stacked bars
- **Performance trend by period:** line chart across review cycles
- **Headcount by gender and level:** stacked bars
- **Top termination reasons:** horizontal bars
- **Salary distribution by department:** range chart
- **Slicers:** department, level, contract type, period, status

## Repo structure

```
veltrix-people-analytics/
├── README.md
├── HR_Analysis.pbix
├── dax_measures.md          — DAX measures with comments
├── data_model.md            — table schemas, relationships, data quality notes
├── power_query.md           — M transformations applied in Power BI
├── python_etl_alternative.md — pandas-based ETL alternative to Power Query
├── dataset/
│   ├── rh_colaboradores.csv
│   ├── rh_avaliacoes.csv
│   └── rh_movimentacoes.csv
└── theme/
    ├── veltrix_theme.json
    └── colors.json
```
