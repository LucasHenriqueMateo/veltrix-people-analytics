# Data Model — Veltrix Group People Analytics

Three tables, related by `colaborador_id` (one-to-many from Employees).

```
Colaboradores (colaborador_id) ──< Avaliacoes (colaborador_id)
Colaboradores (colaborador_id) ──< Movimentacoes (colaborador_id)
```

## Table: Colaboradores (650 rows)

Current employee snapshot.

| Column | Type | Description |
|---|---|---|
| colaborador_id | Text | Unique ID — format `VLT-XXXX` |
| nome | Text | Full name |
| genero | Text | Standardized to `Masculino` / `Feminino` / `Não informado` |
| data_nascimento | Date | Birth date |
| data_admissao | Date | Hire date |
| cargo | Text | Current job title |
| nivel | Text | `Junior` · `Pleno` · `Senior` · `Especialista` · `Gestor` |
| departamento | Text | One of 9 departments |
| salario | Decimal | Monthly salary (BRL) |
| regime | Text | `CLT` or `PJ` |
| status | Text | `Ativo` or `Inativo` |
| cidade / estado | Text | Work location |
| escolaridade | Text | Education level |
| idade | Integer | Computed from birth date |
| tempo_empresa_anos | Integer | Computed from hire date |
| faixa_etaria | Text | Age bracket: up to 25 / 26–35 / 36–45 / 46+ |
| faixa_salarial | Text | Salary bracket: up to R$5k / R$5k–10k / above R$10k |

## Table: Avaliacoes (~2,683 rows)

Performance reviews, quarterly/semi-annual, 2024-Q1 through 2025-Q2.

| Column | Type | Description |
|---|---|---|
| avaliacao_id | Text | Unique ID — format `AVL-XXXXX` |
| colaborador_id | Text | FK → Colaboradores |
| periodo | Text | e.g. `2024-Q1` |
| nota_tecnica | Decimal | Technical score, 1–10 |
| nota_comportamental | Decimal | Behavioral score, 1–10 |
| nota_gestor | Decimal | Manager score, 1–10 — nullable (~8% missing) |
| media_geral | Decimal | Average of the three scores |
| resultado | Text | `Abaixo` · `Dentro` · `Acima` · `Excepcional` do Esperado |

## Table: Movimentacoes (~989 rows)

Full movement history since hiring.

| Column | Type | Description |
|---|---|---|
| movimentacao_id | Text | Unique ID — format `MOV-XXXXX` |
| colaborador_id | Text | FK → Colaboradores |
| tipo | Text | `Admissão` · `Demissão` · `Promoção` · `Transferência` · `Aumento Salarial` |
| data | Date | Movement date |
| motivo | Text | Reason for the movement |
| departamento_origem / destino | Text | Empty on hires / terminations respectively |
| cargo_origem / destino | Text | Job title before and after |
| nivel_origem / destino | Text | Level before and after |
| salario_anterior / novo | Decimal | Salary before and after |
| variacao_salarial_pct | Decimal | Computed % change |

## Data quality issues treated

- **Gender values** — written out fully with inconsistent casing (`Masculino`, `masculino`, `M`); standardized via prefix match
- **Mixed date formats** — ISO and BR formats across all three tables; parsed with day-first heuristic
- **Salary decimal separator** — ~5% of records used comma instead of period; converted before casting to numeric
- **Missing manager scores** — ~8% of reviews have no `nota_gestor`; left null rather than imputed
- **Inconsistent result casing** — review outcomes had typos and casing variants; matched via substring lookup
- **Duplicate movements** — ~12 rows duplicated; removed by `(colaborador_id, tipo, data)` composite key
