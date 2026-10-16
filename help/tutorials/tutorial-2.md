---
title: 'Tutorial 2: controlar seus dados'
description: Saiba como filtrar, agrupar e controlar resultados da consulta em Insights de capacidade de observação.
source-git-commit: 09c8edf64e744ff0c16acd0d5905fc6796490f82
workflow-type: tm+mt
source-wordcount: '336'
ht-degree: 0%
---

## Tutorial 2: controlar seus dados {#tutorial-2-control-your-data}

Neste tutorial, você aprenderá as funções e cláusulas que transformam contagens brutas em respostas: contagens distintas, valores mais recentes, porcentagens, percentis, curingas, classificação e agrupamento multidimensional.

### Contar valores distintos com `uniqueCount`

Quantos hosts e rastreamentos distintos cada serviço tem?

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

`uniqueCount` é exato, não uma estimativa.

Para **listar** os próprios valores distintos (o que o NRQL chama `uniques()`), faceta no atributo:

```sql
SELECT count(*) AS requests FROM Transaction FACET host LIMIT 100
```

### Localizar os valores mais recentes e mais antigos

`latest(attr)` retorna o valor mais recente e `earliest(attr)` o mais antigo. Elas são úteis para tabelas de &quot;estado atual&quot;:

```sql
SELECT latest(value) AS currentCpu FROM CpuUtilization FACET host
```

```sql
SELECT earliest(timestamp) AS firstSeen, latest(timestamp) AS lastSeen FROM Transaction FACET service
```

### Calcular percentuais

`percentage(*, WHERE condition)` retorna o compartilhamento de linhas que correspondem a uma condição. Esta é a taxa de erro:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction
```

Analisar a tendência por serviço:

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO LIMIT 10
```

E rastreie a parcela de solicitações lentas:

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction FACET service
```

### Entender a latência com `percentile`

Médias ocultam valores atípicos. Os percentis mostram o que a maioria dos usuários experimenta (p50) e o que os azarados experimentam (p95, p99):

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

>[!TIP]
>
> As páginas p50, p95 e p99 de `Transaction.duration` são fornecidas a partir de rollups em janelas longas, portanto, permanecem rápidas para visualizações semanais.

### Tornar os resultados legíveis com rótulos

Usar `AS` com aspas simples para rótulos que contenham espaços:

```sql
SELECT count(*) AS 'Requests', percentile(duration, 95) AS 'p95 (ms)' FROM Transaction FACET service
```

>[!NOTE]
>
> Os aliases se aplicam somente a `SELECT` itens. As colunas de facetas são sempre rotuladas com o nome do atributo.

### Combinar padrões e listas

`LIKE` usa `%` para qualquer número de caracteres e `_` para exatamente um:

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath LIKE '/content/%.html' FACET urlPath LIMIT 20
```

Excluir ativos estáticos com `NOT LIKE`:

```sql
SELECT count(*) AS requests FROM Transaction WHERE urlPath NOT LIKE '%.js' AND urlPath NOT LIKE '%.css' AND urlPath NOT LIKE '%.png' FACET urlPath LIMIT 20
```

Corresponder com uma lista com `IN` e `NOT IN` (até 100 valores):

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.method IN ('POST', 'PUT', 'DELETE') FACET service
```

>[!NOTE]
>
> O SQL não tem expressões regulares (`RLIKE`). Combine `LIKE` padrões com `OR` ou use `IN`.

### Classificar e classificar com `ORDER BY`

Por padrão, os resultados facetados são classificados pela primeira agregação. Use `ORDER BY` para classificar de acordo com outra coisa, como as operações mais lentas por p95:

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

Ou os serviços menos usados primeiro:

```sql
SELECT count(*) AS requests FROM Transaction FACET service ORDER BY requests ASC LIMIT 10
```

> **Dica**
>
> Não há `HAVING`. Para localizar apenas operações &quot;ocupadas&quot;, classifique por `requests` e mantenha os N principais, ou adicione um `WHERE` que remova o ruído (por exemplo, `WHERE http.method != 'OPTIONS'`).

### Agrupar por vários atributos

`FACET` aceita até cinco atributos. Cada combinação se torna uma linha ou série:

```sql
SELECT count(*) AS requests FROM Transaction WHERE http.statusCode >= 400 FACET service, http.method, http.statusCode LIMIT 30
```

### Olhar para uma janela anterior

Não há `COMPARE WITH`. Nos painéis e em Explorar, altere o **seletor de hora**. Quando você chama a API sem um intervalo explícito, `SINCE … UNTIL …` fornece qualquer janela passada:

```sql
SELECT count(*) AS lastWeekSameHour FROM Transaction SINCE 169 hours ago UNTIL 168 hours ago
```
