# DAX Measures — Veltrix Group People Analytics

All measures live in a dedicated table `[_Measures]`. Grouped by theme.

---

## Core KPIs

```dax
Active Headcount =
CALCULATE(
    COUNTROWS(hr_Employees),
    hr_Employees[Status] = "Active"
)
```

```dax
Total Hires =
CALCULATE(
    COUNTROWS(hr_JobMovements),
    hr_JobMovements[Type] = "Hire"
)
```

```dax
Total Terminations =
CALCULATE(
    COUNTROWS(hr_JobMovements),
    hr_JobMovements[Type] = "Termination"
)
```

```dax
Turnover Rate =
DIVIDE(
    [Total Terminations],
    [Active Headcount],
    0
)
```

---

## Financial metrics

```dax
-- Active employees only
Total Payroll =
CALCULATE(
    SUM(hr_Employees[Salary]),
    hr_Employees[Status] = "Active"
)
```

```dax
Average Salary =
CALCULATE(
    AVERAGE(hr_Employees[Salary]),
    hr_Employees[Status] = "Active"
)
```

---

## Performance metrics

```dax
Avg Technical Score =
AVERAGE(hr_PerformanceReviews[TechnicalScore])
```

```dax
Avg Behavioral Score =
AVERAGE(hr_PerformanceReviews[BehavioralScore])
```

```dax
Avg Manager Score =
AVERAGE(hr_PerformanceReviews[ManagerScore])
```

```dax
Avg Overall Score =
AVERAGE(hr_PerformanceReviews[OverallAverage])
```

---

## Internal movements

```dax
Total Promotions =
CALCULATE(
    COUNTROWS(hr_JobMovements),
    hr_JobMovements[Type] = "Promotion"
)
```

```dax
Total Transfers =
CALCULATE(
    COUNTROWS(hr_JobMovements),
    hr_JobMovements[Type] = "Transfer"
)
```

```dax
Avg Salary Increase Pct =
AVERAGE(hr_JobMovements[SalaryVariationPct])
```
