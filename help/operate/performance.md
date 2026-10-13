---
title: Desempenho
description: Melhore o desempenho da consulta, do gráfico e do painel em Insights de capacidade de observação.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '227'
ht-degree: 3%
---

## Desempenho {#performance}

Cada consulta é executada com um tempo limite de 15 segundos e um orçamento de leitura de bytes. Este documento explica como o Observability Insights acelera consultas automaticamente e como projetar painéis que permanecem rápidos em qualquer intervalo de tempo.

### Aceleração do rollup (automática)

Os gráficos `Transaction` leem tabelas pré-agregadas quando **todas** destas tabelas são suspensas:

| Condição | Requisito |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Forma | A consulta tem `TIMESERIES` (os totais de Número/Tabela sempre são lidos brutos para exatidão) |
| Janela | ≥ 3 h (nível de 1 min) · ≥ 1 d (nível de 5 min) · ≥ 7 d (nível de 1 h) |
| Agregados | somente `count(*)`, `percentage(*, WHERE status = 'Error')`, `apdex(duration, 500)`, `percentile(duration, 50 \| 95 \| 99)` |
| `FACET` / `WHERE` | somente em `service`, `appInstance`, `kind` |

`CpuUtilization` e `MemoryUtilization` (`average(value)`) e `JvmCpuRecentUtilization` (`average(value)`) gráficos com `FACET host` / `WHERE host …` durante ≥ 3 horas de leitura de rollups de 1 minuto.

**Dica de design:** para widgets de tendências de 7 a 31 dias, siga o conjunto de fácil acúmulo (p95, % de erro, Apdex @ 500, contagem, serviço FACET) — eles permanecem rápidos em qualquer intervalo. Salve os filtros `FACET host`/`name`, `average(duration)` ou `urlPath` para janelas ou tabelas mais curtas.

### Dicas gerais

- Use `LIMIT` em cada consulta facetada — especialmente `FACET urlPath`, `traceId`, `jcr.path` (alta cardinalidade).
- Filtrar antecipadamente com `WHERE` em atributos de baixa cardinalidade (`service`, `kind`, `status`).
- Prefira `Transaction` a `Span` quando precisar apenas de solicitações de entrada - bem menos linhas.
- `uniqueCount` é exato; em atributos de alta cardinalidade em janelas longas, é o agregado mais caro.
- Evite `TIMESERIES MAX`, exceto em janelas de alguns minutos.
- Os widgets de painel armazenam em cache por 30 s; o botão atualizar força uma reexecução.
- Um widget que atinge o tempo limite de 15 segundos ou os erros de orçamento de leitura de bytes que se esgotam — estreite a janela, adicione filtros ou reduza aspectos.
