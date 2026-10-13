---
source-git-commit: d3cd77e213674587152d6d33efa65328f420a547
workflow-type: tm+mt
source-wordcount: '295'
ht-degree: 10%
---
### Exemplos de consulta SQL

Aqui está um exemplo mais completo. Para esta consulta:

- Você escolhe `Transaction` como entidade.
- Você usa `SELECT` para calcular a duração do percentil 95.
- Você agrupa os resultados por serviço com `FACET`.
- Você usa `TIMESERIES AUTO` para plotar os dados em um intervalo dimensionado automaticamente.

```sql
FROM Transaction SELECT percentile(duration, 95) AS p95 FACET service TIMESERIES AUTO
```

Isso produz uma linha por serviço em um gráfico de linha, com os 10 serviços mais ocupados exibidos por padrão.

Clique em cada exemplo abaixo para expandi-lo.

<details>
<summary><strong>Consulta básica de solicitações de entrada</strong></summary>

Tempo médio de resposta, em milissegundos, de todas as solicitações de entrada:

```sql
SELECT average(duration) FROM Transaction
```

</details>

<details>
<summary><strong>Nomes de atributos que contêm pontos</strong></summary>

Nomes com pontos não precisam ser citados:

```sql
SELECT count(*) FROM Transaction WHERE http.statusCode >= 500 FACET http.route
```

</details>

<details>
<summary><strong>A consulta retornou várias colunas</strong></summary>

Vários agregados em uma query se tornam colunas em uma Tabela:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, percentile(duration, 95) AS p95, max(duration) AS maxMs FROM Transaction FACET service
```

</details>

<details>
<summary><strong>Agregados condicionais em uma passagem</strong></summary>

Total de erros e 5xx lado a lado:

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, count(*) FILTER (WHERE http.statusCode >= 500) AS http5xx FROM Transaction FACET service
```

</details>

<details>
<summary><strong>Consultando uma métrica com curadoria</strong></summary>

Média de CPU por host ao longo do tempo:

```sql
SELECT average(value) AS cpu FROM CpuUtilization FACET host TIMESERIES AUTO
```

</details>

<details>
<summary><strong>Consultando qualquer métrica OpenTelemetry por nome</strong></summary>

Usar `MetricGauge` / `MetricSum` para métricas sem uma entidade com curadoria:

```sql
SELECT latest(value) AS bytesUsed FROM MetricSum WHERE metric = 'jvm.memory.used' FACET host, pool
```

</details>

<a id="what-sql-doesnt-do"></a>

### O que o SQL não faz

O SQL é deliberadamente pequeno. Isso mantém as consultas seguras, rápidas e isoladas do locatário. Quando você precisar de um desses recursos, use a alternativa.

| Recurso | Status | Alternativa |
| ------------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------ |
| `SELECT *` | Não suportado | Liste os atributos desejados: `SELECT timestamp, name, duration, traceId …`. |
| Combinação de atributos brutos e agregados | Não suportado | Use `FACET attr` ou `latest(attr)` / `earliest(attr)`. |
| Matemática em `SELECT` (`a - b`, `count(*) / 60`) | Não suportado | Widget (painéis) de **Fórmula**. |
| Associações e subconsultas | Não suportado | Widget de **Fórmula** ou dois widgets lado a lado. |
| `HAVING` | Não suportado | `ORDER BY alias` + `LIMIT`. |
| `COMPARE WITH` | Não suportado | Altere o seletor de hora ou coloque dois widgets em guias diferentes. |
| `FACET CASES` | Não suportado | Uma coluna `count(*) FILTER (WHERE …)` por caso (consulte [Tutorial 3](#83-group-values-into-cases-with-filter)). |
| `histogram()` | Não suportado | `FILTER` colunas em bucket (consulte [Tutorial 3](#84-build-a-latency-histogram)). |
| `uniques()` / `keyset()` | Não suportado | `FACET attr LIMIT n` lista valores distintos. Consultas de descoberta listam métricas e atributos preenchidos. |
| `RLIKE` (regex) | Não suportado | `LIKE` / `NOT LIKE` com os curingas `%` e `_`, ou `IN (…)`. |
| Comentários (`--`, `//`, `/* */`) | Não suportado | Coloque notas no título do widget ou em um widget **Texto**. |
| Datas absolutas em `SINCE` | Não suportado | Use o intervalo absoluto do seletor de tempo. |
| Conversão de tipo (`numeric()`, `string()`) | Não é necessário | Os atributos são digitados no catálogo. |
