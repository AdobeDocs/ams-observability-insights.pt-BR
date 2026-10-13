---
title: 'Tutorial 4: funções avançadas'
description: Saiba como usar funções SQL avançadas para analisar a telemetria do Observability Insights.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '507'
ht-degree: 1%
---

## Tutorial 4: funções avançadas {#tutorial-4-advanced-functions}

Este tutorial aborda as funções e entidades que possibilitam investigações mais profundas: variabilidade, crescimento, pesquisas de &quot;pior exemplo&quot;, compartilhamento do total, composição de transações e métricas arbitrárias de OpenTelemetry.

**Neste tutorial:** [9.1 stddev](#91-measure-consistency-with-stddev) · [9.2 derivativo](#92-spot-growth-with-derivative) · [9.3 Classificado como mais recente](#93-find-the-worst-example-with-ranked-latest) · [9.4 percentOfTotal](#94-find-where-time-goes-with-percentoftotal) · [9.5 SpanBreakdown](#95-break-down-transactions-with-spanbreakdown) · [9.6 Qualquer métrica](#96-discover-and-query-any-metric) · [9.7 Escolher a entidade](#97-choose-the-right-trace-entity) · [9.8 Agregação aninhada](#98-work-around-nested-aggregation)

### Consistência de medida com `stddev`

Duas operações podem compartilhar uma média e ainda se comportar de forma muito diferente. O desvio padrão mostra a variação da latência:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name ORDER BY stddevMs DESC LIMIT 20
```

Um desvio padrão maior que a média aponta para uma operação com solicitações ocasionais muito lentas. Veja a próxima p99.

### Crescimento pontual com `derivative`

`derivative(attr)` retorna a alteração por segundo da média do atributo entre compartimentos. É útil para detectar um medidor que esteja subindo, como o uso de pilha antes de um vazamento:

```sql
SELECT derivative(value) AS heapMBPerSec FROM JvmHeapUsed FACET host TIMESERIES 5 minutes
```

Uma linha positiva sustentada significa crescimento. Valores negativos significam que o indicador está caindo (por exemplo, após a coleta de lixo). Assim como `rate`, `derivative` precisa de `TIMESERIES` e deve ser a única agregação.

### Encontre o pior exemplo com `latest` classificado

O segundo argumento de `latest` e `earliest` altera &quot;mais recente&quot; para &quot;mais alto por este atributo&quot;. `latest(traceId, duration)` retorna a ID de rastreamento da solicitação mais lenta, para que você possa colá-la na pesquisa de rastreamento:

```sql
SELECT count(*) AS requests, max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace, latest(urlPath, duration) AS slowestPath FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 15
```

Use `earliest(traceId, duration)` para obter o exemplo mais rápido para comparação.

### Descobrir para onde o tempo vai com `percentOfTotal`

`percentOfTotal(attr)` mostra a participação de cada grupo no total. Aplicado a `duration`, mostra quais operações consomem mais tempo do servidor. Geralmente, isso é mais útil do que quais operações são mais lentas:

```sql
SELECT count(*) AS requests, sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfServerTime FROM Transaction FACET name LIMIT 15
```

`percentOfTotal` não pode ser combinado com `TIMESERIES`.

### Analisar transações com `SpanBreakdown`

`SpanBreakdown` é pré-agregado ao assimilar. Ele registra quais operações filho (`segment`) cada transação (`rootOperation`) gasta tempo, minuto a minuto. Ela responde &quot;o que esse terminal está fazendo?&quot; barato:

```sql
SELECT sum(callCount) AS calls, average(avgDurationMs) AS avgMs FROM SpanBreakdown WHERE rootOperation LIKE '%GET%' FACET rootOperation, segment LIMIT 25
```

Colocar um segmento em tendência ao longo do tempo:

```sql
SELECT average(avgDurationMs) AS avgMs FROM SpanBreakdown FACET segment TIMESERIES AUTO LIMIT 8
```

>[!NOTE]
>
> `avgDurationMs` não está marcado automaticamente com uma unidade. Defina o widget **Unidade** como `ms`.

### Detectar e consultar qualquer métrica

O SQL não tem `keyset()` ou `SHOW EVENT TYPES`. Em vez disso, liste cada métrica que está relatando, juntamente com sua unidade e valor mais recente:

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricGauge FACET metric, unit LIMIT 200
```

```sql
SELECT count(*) AS samples, latest(value) AS latestValue FROM MetricSum FACET metric, unit LIMIT 200
```

Em seguida, consulte qualquer métrica por nome. Por exemplo, memória JVM usada, por pool:

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool LIMIT 50
```

Para ver quais **atributos span** são preenchidos para cada serviço, conte os valores não vazios:

```sql
SELECT count(*) FILTER (WHERE http.route != '') AS withRoute, count(*) FILTER (WHERE db.system != '') AS dbCalls, count(*) FILTER (WHERE peer.service != '') AS withPeer, count(*) FILTER (WHERE error.type != '') AS withErrorType FROM Span FACET service LIMIT 50
```

>[!TIP]
>
> Quando uma entidade com curadoria existe (por exemplo, `CpuUtilization`), prefira-a a `MetricGauge`. Ela aplica o filtro correto e a conversão de unidade, e pode ser acelerada por rollup.

### Escolher a entidade de rastreamento correta

| Usar | Quando quiser | Exemplo |
| ------------- | ------------------------------------------------------------------------------------------- | --------------------------------- |
| `Transaction` | KPIs de solicitação HTTP/RPC de entrada (taxa de transferência, latência, erros, Apdex). Acelerado por rollup. | Sinais dourados |
| `AppRequest` | Os mesmos KPIs, **incluindo consumidores de mensagens** (Kafka e semelhante) | Serviços orientados por eventos |
| `Span` | Qualquer coisa abaixo da solicitação: HTTP de saída, DB, mensagens, AEM Sling/JCR/JSP, extensões internas | Análise de dependência e caminho de código |

Conte cada tipo de span para ver o que é instrumentado:

```sql
SELECT count(*) AS spans, percentile(duration, 95) AS p95 FROM Span FACET kind
```

Chamadas de saída por domínio externo:

```sql
SELECT count(*) AS calls, percentage(*, WHERE status = 'Error') AS errorPct, percentile(duration, 95) AS p95 FROM Span WHERE kind = 'Client' AND externalDomain != '' FACET externalDomain LIMIT 20
```

### Contornar a agregação aninhada

O NRQL permite agregar uma agregação com uma subconsulta, por exemplo &quot;a média dos máximos por host&quot;. O SQL não. Estas são as alternativas:

| Objetivo | Abordagem SQL |
| --------------------------------------- | ---------------------------------------------------------- |
| Um valor por grupo e classifique os grupos | `FACET group` + `ORDER BY alias` + `LIMIT` |
| Uma proporção de dois agregados | [Widget de fórmula](#86-combine-queries-with-formula-widgets) |
| Um valor por host, depois o &quot;pior host&quot; | `FACET host ORDER BY alias DESC LIMIT 1` |
| Uma média por host ao longo do tempo | `FACET host TIMESERIES AUTO` e leia a série |

Exemplo: o pior host por p95:

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction FACET host ORDER BY p95 DESC LIMIT 1
```
