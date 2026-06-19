# Power Query Transformations — Veltrix Group People Analytics

Three separate queries, one per source table. Applied in the Advanced Editor after importing each CSV.

---

## Query 1: Colaboradores

1. **Standardize gender** — raw values ranged from `"M"` to `"masculino"` to `"Masculino"`. Matched by uppercased prefix and mapped to `Masculino` / `Feminino` / `Não informado`.

2. **Parse dates** — `data_nascimento` and `data_admissao` arrived in both ISO and BR formats. A custom function detects BR format when the first segment exceeds 12.

3. **Fix salary decimal separator** — ~5% of rows used a comma instead of a period; replaced before casting to numeric.

4. **Add age** — computed from `data_nascimento` against the current date, rounded down to whole years.

5. **Add tenure (years)** — same approach using `data_admissao`.

6. **Add age bracket** — buckets: up to 25, 26–35, 36–45, 46+.

7. **Add salary bracket** — buckets: up to R$5k, R$5k–10k, above R$10k.

---

## Query 2: Avaliações

1. **Standardize result labels** — raw values had casing and spelling inconsistencies. Matched by lowercased substring (`"abaixo"`, `"dentro"`, `"acima"`, `"excep"`) and mapped to canonical labels.

2. **Cast scores to numeric** — `nota_tecnica`, `nota_comportamental`, `nota_gestor`, `media_geral` — nulls preserved where the score wasn't given (mainly `nota_gestor`).

3. **Split period into year and quarter** — `periodo` (e.g. `2024-Q1`) parsed into separate `ano` and `trimestre` columns for easier slicing.

---

## Query 3: Movimentações

1. **Remove duplicates** — ~12 rows duplicated; removed using the composite key `(colaborador_id, tipo, data)`, since `movimentacao_id` alone wasn't reliable.

2. **Parse dates** — same BR/ISO detection logic as Query 1.

3. **Fix salary decimal separator** — applied to both `salario_anterior` and `salario_novo`.

4. **Add salary variation %** — `(salario_novo - salario_anterior) / salario_anterior`, null when there's no prior salary to compare against.

5. **Add year and month** — extracted from `data` for the time-series visuals.

---

## Note on the Python alternative

`python_etl_alternative.md` contains a `pandas`-based script that performs the same cleaning, plus translates all column names and categorical values to English. It wasn't used to feed the actual Power BI model (which runs on Power Query M with Portuguese field names) — it's included as a reference for how the same ETL would look in a Python/pandas pipeline.
