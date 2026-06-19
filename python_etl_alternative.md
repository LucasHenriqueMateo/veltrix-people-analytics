# Python ETL Alternative — pandas pipeline

The Power BI model is built on Power Query (M), documented in `power_query.md`. This file shows the same cleaning logic reimplemented in `pandas`, for cases where the pipeline needs to run outside Power BI — a scheduled job, a data warehouse load, or a notebook.

It applies the same transformations as the M queries, plus translates every column name and categorical value to English.

## What it does differently from the M version

- Renames all columns to English (`colaborador_id` → `EmployeeID`, `genero` → `Gender`, etc.)
- Translates categorical values (`Ativo` → `Active`, `Demissão` → `Termination`, `Excepcional` → `Exceptional`)
- Computes the same derived columns (age, tenure, brackets) using vectorized `numpy` conditions instead of M's `if/else` chains

## Script

```python
import pandas as pd
import numpy as np
from datetime import datetime

FOLDER_PATH = r"<path-to-csv-folder>"

# ── Employees ──────────────────────────────────────────────────
df_emp = pd.read_csv(f"{FOLDER_PATH}\\rh_colaboradores.csv", sep=",", encoding="utf-8")

emp_cols = {
    "colaborador_id": "EmployeeID", "nome": "Name", "genero": "Gender",
    "data_nascimento": "BirthDate", "data_admissao": "HireDate", "cargo": "JobTitle",
    "nivel": "Level", "departamento": "Department", "salario": "Salary",
    "regime": "ContractType", "status": "Status", "cidade": "City",
    "estado": "State", "escolaridade": "EducationLevel"
}
df_emp.rename(columns=emp_cols, inplace=True)

conditions_gender = [
    df_emp['Gender'].str.upper().str.startswith('M', na=False),
    df_emp['Gender'].str.upper().str.startswith('F', na=False)
]
df_emp['Gender'] = np.select(conditions_gender, ['Male', 'Female'], default='Not Disclosed')

df_emp['BirthDate'] = pd.to_datetime(df_emp['BirthDate'], format='mixed', dayfirst=True, errors='coerce')
df_emp['HireDate'] = pd.to_datetime(df_emp['HireDate'], format='mixed', dayfirst=True, errors='coerce')

df_emp['Salary'] = df_emp['Salary'].astype(str).str.replace(',', '.').replace('nan', np.nan)
df_emp['Salary'] = pd.to_numeric(df_emp['Salary'], errors='coerce')

current_date = pd.to_datetime(datetime.now().date())
df_emp['Age'] = np.floor((current_date - df_emp['BirthDate']).dt.days / 365.25)
df_emp['TenureYears'] = np.floor((current_date - df_emp['HireDate']).dt.days / 365.25)

conditions_age = [
    df_emp['Age'].isna(), df_emp['Age'] <= 25, df_emp['Age'] <= 35, df_emp['Age'] <= 45
]
df_emp['AgeGroup'] = np.select(conditions_age, ['Not Disclosed', 'Under 25', '26–35', '36–45'], default='46+')

conditions_salary = [
    df_emp['Salary'].isna(), df_emp['Salary'] <= 5000, df_emp['Salary'] <= 10000
]
df_emp['SalaryBand'] = np.select(conditions_salary, ['Not Disclosed', 'Up to $5k', '$5k–$10k'], default='Above $10k')

df_emp['Status'] = df_emp['Status'].replace({'Ativo': 'Active', 'Inativo': 'Inactive'})


# ── Performance reviews ────────────────────────────────────────
df_rev = pd.read_csv(f"{FOLDER_PATH}\\rh_avaliacoes.csv", sep=",", encoding="utf-8")

rev_cols = {
    "avaliacao_id": "ReviewID", "colaborador_id": "EmployeeID", "periodo": "Period",
    "nota_tecnica": "TechnicalScore", "nota_comportamental": "BehavioralScore",
    "nota_gestor": "ManagerScore", "media_geral": "OverallAverage", "resultado": "Result"
}
df_rev.rename(columns=rev_cols, inplace=True)

res_lower = df_rev['Result'].str.lower()
conditions_res = [
    res_lower.str.contains('abaixo', na=False), res_lower.str.contains('dentro', na=False),
    res_lower.str.contains('acima', na=False), res_lower.str.contains('excep', na=False)
]
choices_res = ['Below Expectations', 'Meets Expectations', 'Exceeds Expectations', 'Exceptional']
df_rev['Result'] = np.select(conditions_res, choices_res, default=df_rev['Result'])

for col in ['TechnicalScore', 'BehavioralScore', 'ManagerScore', 'OverallAverage']:
    df_rev[col] = pd.to_numeric(df_rev[col], errors='coerce')

df_rev['Year'] = df_rev['Period'].str[:4].astype('Int64')
df_rev['Quarter'] = df_rev['Period'].str[-2:]


# ── Job movements ──────────────────────────────────────────────
df_mov = pd.read_csv(f"{FOLDER_PATH}\\rh_movimentacoes.csv", sep=",", encoding="utf-8")

mov_cols = {
    "movimentacao_id": "MovementID", "colaborador_id": "EmployeeID", "tipo": "Type",
    "data": "Date", "motivo": "Reason", "departamento_origem": "OriginDepartment",
    "departamento_destino": "DestinationDepartment", "cargo_origem": "OriginJobTitle",
    "cargo_destino": "DestinationJobTitle", "nivel_origem": "OriginLevel",
    "nivel_destino": "DestinationLevel", "salario_anterior": "PreviousSalary",
    "salario_novo": "NewSalary"
}
df_mov.rename(columns=mov_cols, inplace=True)

df_mov.drop_duplicates(subset=['EmployeeID', 'Type', 'Date'], inplace=True)

type_lower = df_mov['Type'].str.lower()
conditions_type = [
    type_lower.str.contains('admissão', na=False), type_lower.str.contains('demissão', na=False),
    type_lower.str.contains('promoção', na=False), type_lower.str.contains('transferência', na=False),
    type_lower.str.contains('aumento', na=False)
]
choices_type = ['Hire', 'Termination', 'Promotion', 'Transfer', 'Salary Increase']
df_mov['Type'] = np.select(conditions_type, choices_type, default=df_mov['Type'])

df_mov['Date'] = pd.to_datetime(df_mov['Date'], format='mixed', dayfirst=True, errors='coerce')

for col in ['PreviousSalary', 'NewSalary']:
    df_mov[col] = df_mov[col].astype(str).str.replace(',', '.').replace('nan', np.nan)
    df_mov[col] = pd.to_numeric(df_mov[col], errors='coerce')

df_mov['SalaryVariationPct'] = np.where(
    (df_mov['PreviousSalary'].isna()) | (df_mov['PreviousSalary'] == 0),
    np.nan,
    (df_mov['NewSalary'] - df_mov['PreviousSalary']) / df_mov['PreviousSalary']
)

df_mov['Year'] = df_mov['Date'].dt.year.astype('Int64')
df_mov['Month'] = df_mov['Date'].dt.month.astype('Int64')
```
