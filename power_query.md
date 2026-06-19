# Power Query Transformations — Veltrix Group People Analytics

Source data arrives in Portuguese (`colaborador_id`, `genero`, `salario`...). Before loading into the model, Power Query translates column names and categorical values to English and applies the same cleaning logic as the `pandas` alternative documented in `python_etl_alternative.md`.

---

## hr_Employees

1. **Rename columns** to English (`colaborador_id` → `EmployeeID`, `genero` → `Gender`, `salario` → `Salary`, etc.)

2. **Standardize gender** — raw values ranged from `"M"` to `"masculino"` to `"Masculino"`. Matched by uppercased prefix and mapped to `Male` / `Female` / `Not Disclosed`.

3. **Parse dates** — `BirthDate` and `HireDate` arrived in both ISO and BR formats. A custom function detects BR format when the day segment exceeds 12.

4. **Fix salary decimal separator** — ~5% of rows used a comma instead of a period; replaced before casting to numeric.

5. **Add Age** — computed from `BirthDate` against the current date, rounded down to whole years.

6. **Add TenureYears** — same approach using `HireDate`.

7. **Add AgeGroup** — buckets: Under 25, 26–35, 36–45, 46+.

8. **Add SalaryBand** — buckets: Up to $5k, $5k–10k, Above $10k.

9. **Translate Status** — `Ativo` → `Active`, `Inativo` → `Inactive`.

---

## hr_PerformanceReviews

1. **Rename columns** to English (`avaliacao_id` → `ReviewID`, `nota_tecnica` → `TechnicalScore`, etc.)

2. **Standardize and translate Result** — raw values had casing and spelling inconsistencies. Matched by lowercased substring and mapped to `Below Expectations` / `Meets Expectations` / `Exceeds Expectations` / `Exceptional`.

3. **Add ResultOrder** — integer sort key so visuals show results in logical order rather than alphabetical.

4. **Cast scores to numeric** — nulls preserved where the score wasn't given (mainly `ManagerScore`).

5. **Split Period into Year and Quarter** — `Period` (e.g. `2024-Q1`) parsed into separate columns for easier slicing.

---

## hr_JobMovements

1. **Rename columns** to English (`movimentacao_id` → `MovementID`, `tipo` → `Type`, etc.)

2. **Remove duplicates** — ~12 rows duplicated; removed using the composite key `(EmployeeID, Type, Date)`.

3. **Translate Type** — `Admissão` → `Hire`, `Demissão` → `Termination`, `Promoção` → `Promotion`, `Transferência` → `Transfer`, `Aumento Salarial` → `Salary Increase`.

4. **Parse dates** — same BR/ISO detection logic.

5. **Fix salary decimal separator** — applied to both `PreviousSalary` and `NewSalary`.

6. **Add SalaryVariationPct** — `(NewSalary - PreviousSalary) / PreviousSalary`, null when there's no prior salary to compare against.

7. **Add Year and Month** — extracted from `Date` for the time-series visuals.

---

## dCalendar

A standalone date dimension table, generated independently rather than derived from any fact table. This lets the model relate dates consistently across `hr_JobMovements` and any future fact table without each one needing its own date logic — the standard "star schema" pattern for time intelligence in Power BI.

---

## Note on the Python alternative

`python_etl_alternative.md` documents the same logic in `pandas`. It was used as the reference implementation for naming and translation — the Power Query steps above mirror it column-for-column.
