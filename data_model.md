# Data Model — Veltrix Group People Analytics

Four tables: three fact/dimension tables plus a dedicated calendar table for time intelligence.

```
hr_Employees (EmployeeID) ──< hr_PerformanceReviews (EmployeeID)
hr_Employees (EmployeeID) ──< hr_JobMovements (EmployeeID)
dCalendar (Date)           ──< hr_JobMovements (Date)
```

## Table: hr_Employees (650 rows)

Current employee snapshot.

| Column | Type | Description |
|---|---|---|
| EmployeeID | Text | Unique ID — format `VLT-XXXX` |
| Name | Text | Full name |
| Gender | Text | `Male` / `Female` / `Not Disclosed` |
| BirthDate | Date | Birth date |
| HireDate | Date | Hire date |
| JobTitle | Text | Current job title |
| Level | Text | `Junior` · `Pleno` · `Senior` · `Especialista` · `Gestor` |
| Department | Text | One of 9 departments |
| Salary | Decimal | Monthly salary (BRL) |
| ContractType | Text | `CLT` or `PJ` |
| Status | Text | `Active` or `Inactive` |
| City / State | Text | Work location |
| EducationLevel | Text | Education level |
| Age | Integer | Computed from birth date |
| AgeGroup | Text | `Under 25` · `26–35` · `36–45` · `46+` |
| TenureYears | Integer | Computed from hire date |
| SalaryBand | Text | `Up to $5k` · `$5k–$10k` · `Above $10k` |

## Table: hr_PerformanceReviews (~2,683 rows)

Performance reviews, quarterly/semi-annual, 2024-Q1 through 2025-Q2.

| Column | Type | Description |
|---|---|---|
| ReviewID | Text | Unique ID — format `AVL-XXXXX` |
| EmployeeID | Text | FK → hr_Employees |
| Period | Text | e.g. `2024-Q1` |
| TechnicalScore | Decimal | 1–10 |
| BehavioralScore | Decimal | 1–10 |
| ManagerScore | Decimal | 1–10 — nullable (~8% missing) |
| OverallAverage | Decimal | Average of the three scores |
| Result | Text | `Below Expectations` · `Meets Expectations` · `Exceeds Expectations` · `Exceptional` |
| ResultOrder | Integer | Sort key for `Result`, so charts order correctly |
| Year / Quarter | Integer / Text | Parsed from `Period` |

## Table: hr_JobMovements (~989 rows)

Full movement history since hiring.

| Column | Type | Description |
|---|---|---|
| MovementID | Text | Unique ID — format `MOV-XXXXX` |
| EmployeeID | Text | FK → hr_Employees |
| Type | Text | `Hire` · `Termination` · `Promotion` · `Transfer` · `Salary Increase` |
| Date | Date | Movement date — FK → dCalendar |
| Reason | Text | Reason for the movement |
| OriginDepartment / DestinationDepartment | Text | Empty on hires / terminations respectively |
| OriginJobTitle / DestinationJobTitle | Text | Job title before and after |
| OriginLevel / DestinationLevel | Text | Level before and after |
| PreviousSalary / NewSalary | Decimal | Salary before and after |
| SalaryVariationPct | Decimal | Computed % change |
| Year / Month | Integer | Extracted from `Date` |

## Table: dCalendar

Standalone date dimension, built for clean time-intelligence behavior — avoids relying on `hr_JobMovements[Date]` directly for slicing across multiple fact tables.

| Column | Type | Description |
|---|---|---|
| Date | Date | One row per day in the covered range |
| Year | Integer | Calendar year |
| Quarter | Text | `Q1`–`Q4` |
| Month Number | Integer | 1–12 |
| Month Name / Month Abbr | Text | Full and abbreviated month name |
| Month Year / Year-Month | Text | Combined labels for axis sorting |

## Data quality issues treated

- **Gender values** — written out fully with inconsistent casing in the original source; standardized to `Male` / `Female` / `Not Disclosed`
- **Mixed date formats** — ISO and BR formats across all three tables; parsed with day-first heuristic
- **Salary decimal separator** — ~5% of records used comma instead of period; converted before casting to numeric
- **Missing manager scores** — ~8% of reviews have no `ManagerScore`; left null rather than imputed
- **Inconsistent result casing** — review outcomes had typos and casing variants; matched via substring lookup
- **Duplicate movements** — ~12 rows duplicated; removed by `(EmployeeID, Type, Date)` composite key
