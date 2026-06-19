# DAX Measures — Veltrix Group People Analytics

All measures live in a dedicated table `[_Medidas_RH]`. Grouped by theme.

---

## Headcount

```dax
Headcount Total =
CALCULATE(COUNTROWS(Colaboradores), Colaboradores[status] = "Ativo")
```

```dax
Headcount Inativo =
CALCULATE(COUNTROWS(Colaboradores), Colaboradores[status] = "Inativo")
```

```dax
% CLT =
DIVIDE(
    CALCULATE([Headcount Total], Colaboradores[regime] = "CLT"),
    [Headcount Total], BLANK()
)
```

```dax
% PJ =
DIVIDE(
    CALCULATE([Headcount Total], Colaboradores[regime] = "PJ"),
    [Headcount Total], BLANK()
)
```

```dax
% Feminino =
DIVIDE(
    CALCULATE([Headcount Total], Colaboradores[genero] = "Feminino"),
    [Headcount Total], BLANK()
)
```

---

## Salary

```dax
-- Active employees only
Salário Médio =
AVERAGEX(
    FILTER(Colaboradores, Colaboradores[status] = "Ativo"),
    Colaboradores[salario]
)
```

```dax
Massa Salarial =
SUMX(
    FILTER(Colaboradores, Colaboradores[status] = "Ativo"),
    Colaboradores[salario]
)
```

```dax
Salário Mediano =
MEDIANX(
    FILTER(Colaboradores, Colaboradores[status] = "Ativo"),
    Colaboradores[salario]
)
```

---

## Turnover

```dax
Total Demissões =
CALCULATE(COUNTROWS(Movimentacoes), Movimentacoes[tipo] = "Demissão")
```

```dax
Total Admissões =
CALCULATE(COUNTROWS(Movimentacoes), Movimentacoes[tipo] = "Admissão")
```

```dax
-- Classic formula: terminations / average headcount (start + end / 2)
Turnover % =
DIVIDE(
    [Total Demissões],
    ([Headcount Total] + [Headcount Inativo]) / 2,
    BLANK()
)
```

```dax
Pedido Colaborador % =
DIVIDE(
    CALCULATE(COUNTROWS(Movimentacoes),
        Movimentacoes[tipo] = "Demissão",
        Movimentacoes[motivo] = "Pedido do colaborador"
    ),
    [Total Demissões], BLANK()
)
```

---

## Movements

```dax
Total Promoções =
CALCULATE(COUNTROWS(Movimentacoes), Movimentacoes[tipo] = "Promoção")
```

```dax
Total Transferências =
CALCULATE(COUNTROWS(Movimentacoes), Movimentacoes[tipo] = "Transferência")
```

```dax
Total Aumentos =
CALCULATE(COUNTROWS(Movimentacoes), Movimentacoes[tipo] = "Aumento Salarial")
```

```dax
-- Average salary bump from promotions and raises combined
Variação Salarial Média % =
AVERAGEX(
    FILTER(Movimentacoes,
        Movimentacoes[tipo] IN {"Promoção","Aumento Salarial"}
        && NOT(ISBLANK(Movimentacoes[variacao_salarial_pct]))
    ),
    Movimentacoes[variacao_salarial_pct]
)
```

---

## Performance

```dax
Nota Média Geral =
AVERAGEX(
    FILTER(Avaliacoes, NOT(ISBLANK(Avaliacoes[media_geral]))),
    Avaliacoes[media_geral]
)
```

```dax
Nota Média Técnica =
AVERAGEX(FILTER(Avaliacoes, NOT(ISBLANK(Avaliacoes[nota_tecnica]))), Avaliacoes[nota_tecnica])
```

```dax
Nota Média Comportamental =
AVERAGEX(FILTER(Avaliacoes, NOT(ISBLANK(Avaliacoes[nota_comportamental]))), Avaliacoes[nota_comportamental])
```

```dax
Total Avaliações =
COUNTROWS(Avaliacoes)
```

```dax
% Resultado Excepcional =
DIVIDE(
    CALCULATE([Total Avaliações], Avaliacoes[resultado] = "Excepcional"),
    [Total Avaliações], BLANK()
)
```

```dax
% Abaixo do Esperado =
DIVIDE(
    CALCULATE([Total Avaliações], Avaliacoes[resultado] = "Abaixo do Esperado"),
    [Total Avaliações], BLANK()
)
```

```dax
-- Active employees with no review on record — coverage gap
Colaboradores sem Avaliação =
CALCULATE(
    COUNTROWS(Colaboradores),
    Colaboradores[status] = "Ativo",
    ISEMPTY(RELATEDTABLE(Avaliacoes))
)
```

---

## Tenure

```dax
Tempo Médio de Empresa (anos) =
AVERAGEX(
    FILTER(Colaboradores, Colaboradores[status] = "Ativo"),
    Colaboradores[tempo_empresa_anos]
)
```

```dax
Tempo Médio até Demissão (anos) =
AVERAGEX(
    FILTER(Colaboradores, Colaboradores[status] = "Inativo"),
    Colaboradores[tempo_empresa_anos]
)
```
