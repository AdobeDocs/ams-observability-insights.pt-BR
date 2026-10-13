---
source-git-commit: 4949fcad799260aa73354a8c0b9d79e6394cb257
workflow-type: tm+mt
source-wordcount: '1479'
ht-degree: 9%
---
## Referência SQL {#sql-reference}

Essa referência abrange sintaxe SQL, cláusulas, funções e operadores. Expanda cada entrada para obter detalhes e exemplos.

**Neste documento:** [Componentes de consulta](#41-query-components) · [Cláusulas obrigatórias](#42-required-clauses) · [Cláusulas opcionais](#43-optional-clauses) · [Funções de agregador](#44-aggregator-functions) · [Operadores e condições](#45-operators-and-conditions) · [Manuseio de tipo](#46-type-handling-blank-vs-null) · [SQL para SQL](#47-translate-sql-to-sql) · [Importação de painéis existentes](#48-importing-existing-dashboards-to-insights)

### Componentes do query

```text
SELECT <select-item> [, <select-item> ...]
FROM   <Entity>
[WHERE      <condition>]
[FACET      <attribute> [, <attribute> ...]]          -- up to 5
[SINCE      <n> <unit> [AGO]]
[UNTIL      NOW | <n> <unit> [AGO]]
[TIMESERIES AUTO | MAX | <n> <unit>]
[ORDER BY   <alias-or-selected-name> [ASC | DESC]]
[LIMIT      <1..5000>]

<select-item> := <function>(<args>) [FILTER (WHERE <condition>)] [AS <alias>]
               | <attribute> [AS <alias>]
<unit>        := second | seconds | minute | minutes | hour | hours | day | days | week | weeks
```

> [!NOTE]
> As `--` anotações acima são somente documentação. O próprio SQL não aceita comentários.

### Cláusulas obrigatórias

<details>
<summary><code>SELECT</code> — o que retornar</summary>

`SELECT` lista **atributos brutos** (uma consulta de evento) ou **funções de agregador** (uma consulta de agregação). Você não pode misturar os dois.

| Formulário | Exemplo | Resultado |
| ------------ | -------------------------------------------------- | ------------------------------------------------- |
| Atributos | `SELECT timestamp, name, duration` | Linhas de eventos individuais (padrão 100) |
| Agregados | `SELECT count(*), average(duration)` | Um valor por agregação (por faceta/período) |
| Com aliases | `SELECT count(*) AS requests` | Rótulo da coluna `requests` |
| Alias entre aspas | `SELECT percentile(duration, 95) AS 'p95 latency'` | Rótulo com espaços |

Lista de eventos:

```sql
SELECT timestamp, service, name, duration, status, traceId FROM Transaction WHERE status = 'Error' LIMIT 50
```

Vários agregados:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

Regras:

- Não há suporte para `SELECT *`. Liste os atributos desejados.
- Não há suporte para correspondência entre atributos ou agregações (`sum(a) / count(*)`). Use um [widget de fórmula](#86-combine-queries-with-formula-widgets).
- A combinação de atributos e agregações falha com _atributos brutos não podem ser combinados com agregações; use FACET para agrupar resultados_.

</details>

<details>
<summary><code>FROM</code> — qual entidade consultar</summary>

`FROM` nomeia exatamente **uma** entidade do [Dicionário de dados](#data-dictionary). O nome da entidade não diferencia maiúsculas de minúsculas.

```sql
SELECT count(*) FROM Transaction
```

```sql
FROM Span SELECT count(*) FACET kind
```

Cada entidade tem um escopo fixo. Por exemplo, `Transaction` é sempre a extensão do servidor e `CpuUtilization` é sempre a métrica de CPU ocupado com `value` convertido em porcentagem. Você não pode consultar mais de uma entidade em uma única consulta. Para combinar entidades, use um [widget de Fórmula](#86-combine-queries-with-formula-widgets).

</details>

### Cláusulas opcionais

<details>
<summary><code>AS</code> — rotular uma coluna</summary>

Use `AS` para dar um nome legível a uma agregação ou atributo. O alias se torna o cabeçalho da coluna, o rótulo da legenda e o nome que você referencia em `ORDER BY`.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET service ORDER BY p95 DESC LIMIT 10
```

Use uma cadeia de caracteres entre aspas simples para rótulos com espaços:

```sql
SELECT percentile(duration, 95) AS 'p95 latency (ms)' FROM Transaction TIMESERIES AUTO
```

</details>

<details>
<summary><code>FACET</code> — resultados do grupo</summary>

`FACET` agrupa resultados por até **5** atributos, como SQL `GROUP BY`.

```sql
SELECT count(*) FROM Transaction FACET service, http.method, http.statusCode LIMIT 30
```

| Consulta | O que `LIMIT n` significa | Padrão |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| `FACET` sem `TIMESERIES` | Número de linhas | 10 |
| `FACET` com `TIMESERIES` | **N primeiras séries**, classificadas pela soma da primeira agregação em toda a janela. Cada período é mantido para essas séries. | Todas as séries (limitadas a 5.000 linhas) |

Séries cronológicas facetadas, top 5:

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Os valores de facetas podem ser sequências ou números. Sequências de caracteres em branco formam seu próprio grupo. Excluí-los com `WHERE attr != ''`.

</details>

<details>
<summary><code>FILTER</code> — agregações condicionais</summary>

Anexe `FILTER (WHERE condition)` a qualquer agregação para calculá-la sobre um subconjunto de linhas. Isso permite comparar várias condições em uma única passagem.

```sql
SELECT count(*) AS total, count(*) FILTER (WHERE status = 'Error') AS errors, percentile(duration, 95) FILTER (WHERE http.method = 'GET') AS getP95 FROM Transaction FACET service LIMIT 20
```

`FILTER` não é permitido com `apdex`, `rate` ou `derivative`. `percentage` já assume sua própria condição.

</details>

<details>
<summary><code>LIMIT</code> — quantas linhas ou séries</summary>

`LIMIT` toma um inteiro positivo. Valores acima de 5.000 são fixados em 5.000.

| Forma do resultado | Padrão |
| ------------ | ---------- |
| Outdoor | 1 |
| Faceta | 10 |
| Séries cronológicas | 5.000 linhas |
| Eventos | 100 |

```sql
SELECT count(*) FROM Transaction FACET urlPath LIMIT 50
```

> [!TIP]
> Sempre defina um `LIMIT` em variáveis orientadas por consulta e em aspectos de alta cardinalidade (`urlPath`, `traceId`, `jcr.path`, `url`).

</details>

<details>
<summary><code>ORDER BY</code> — classificar resultados</summary>

`ORDER BY` classifica por um **alias ou atributo selecionado**. A direção padrão é `DESC`.

```sql
SELECT count(*) AS requests, percentile(duration, 95) AS p95 FROM Transaction FACET name ORDER BY p95 DESC LIMIT 20
```

| Forma do resultado | Ordem padrão |
| ----------------- | --------------------------------------------------------------------------- |
| Outdoor / faceta | Primeira agregação, `DESC` |
| Séries cronológicas | Hora, `ASC` |
| Eventos | Primeira coluna selecionada, `DESC` (assim, coloque `timestamp` primeiro como &quot;mais recente primeiro&quot;) |

Regras:

- A referência a algo que não está selecionado falha com _ORDER BY &#39;x&#39; deve fazer referência a um valor ou alias selecionado_.
- Não adicione `ORDER BY` a uma consulta `TIMESERIES`. Ele substitui a ordem de tempo que os gráficos precisam.

</details>

<details>
<summary><code>SINCE</code> — início da janela</summary>

`SINCE n unit [AGO]` define até que ponto a janela começa. Unidades: `second`, `minute`, `hour`, `day`, `week` (singular ou plural). `AGO` é opcional.

```sql
SELECT count(*) FROM Transaction SINCE 6 hours ago
```

- O padrão é `SINCE 1 hour ago`.
- A janela não pode exceder **31 dias** (_O intervalo de tempo não pode exceder 31 dias_).
- Datas absolutas (`SINCE '2026-01-01'`) e `TODAY` / `YESTERDAY` não são suportadas.

> [!IMPORTANT]
> Nos painéis e em Explorar, o **seletor de tempo substitui** `SINCE` e `UNTIL`. Consulte [Prioridade de janela de tempo](#time-window-precedence).

</details>

<details>
<summary><code>TIMESERIES</code> — plotar ao longo do tempo</summary>

`TIMESERIES` retorna uma linha por intervalo de tempo (por valor de faceta), que está pronta para gráficos de Linha, Área ou Barra.

| Formulário | Tamanho do bloco |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `TIMESERIES 5 minutes` | Exatamente o que você solicita (qualquer unidade) |
| `TIMESERIES AUTO` | Sobre a janela ÷ 120, arredondada para 1s / 5s / 10s / 15s / 30s / 1m / 5m / 10m / 15m / 30m / 1h / 6h / 12h / 1d. **Nunca abaixo de 2 minutos.** |
| `TIMESERIES MAX` | Intervalos de 1 segundo. Use somente em alguns minutos. |

`AUTO` compartimentos típicos:

| Janela | 1 h | 6 h | 24 h | 7 d | 30 d |
| ------ | ----- | ----- | ------ | --- | ---- |
| Balde | 2 min | 5 min | 15 min | 3 h | 6 h |

```sql
SELECT percentile(duration, 95) AS p95 FROM Transaction TIMESERIES AUTO
```

```sql
SELECT count(*) FROM Transaction TIMESERIES 1 hour SINCE 1 day ago
```

> [!TIP]
> Prefira `AUTO` nos painéis. Mantém cerca de 120 pontos o que quer que o seletor de tempo diga. Também evita linhas &quot;quebradas&quot; por host, pois as métricas do host são raspadas a cada 90 segundos.

</details>

<details>
<summary><code>UNTIL</code> — fim da janela</summary>

`UNTIL NOW` (padrão) ou `UNTIL n unit [AGO]`. Combine-o com `SINCE` para ver uma janela anterior:

```sql
SELECT count(*) AS yesterday FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

`SINCE` deve ser anterior a `UNTIL`. Caso contrário, a consulta falha com _SINCE deve ser anterior a UNTIL_.

</details>

<details>
<summary><code>WHERE</code> — filtrar linhas</summary>

`WHERE` mantém somente linhas que correspondem a uma condição. Combine condições com `AND`, `OR`, `NOT` e parênteses. Consulte [Operadores e condições](#45-operators-and-conditions).

```sql
SELECT count(*) FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR http.statusCode >= 500)
```

`WHERE` pode aparecer apenas uma vez. Coloque todas as condições nele.

</details>

### Funções agregadoras

| Função | Argumentos | Devoluções | Funciona em |
| -------------------------------------- | ----------------------------------- | ---------------------- | ------------------------------------- |
| [`apdex`](#fn-apdex) | duração numérica, limite `T` | 0-1 | Rastrear entidades |
| [`average` / `avg`](#fn-average) | numérico | Mesma unidade que a entrada | Qualquer |
| [`count`](#fn-count) | `*` ou atributo | Número de linhas | Qualquer |
| [`derivative`](#fn-derivative) | numérico | Alteração por segundo | Qualquer um (precisa de `TIMESERIES`) |
| [`earliest`](#fn-latest) | qualquer [, classificação] | Valor do atributo | Qualquer |
| [`latest`](#fn-latest) | qualquer [, classificação] | Valor do atributo | Qualquer |
| [`max` / `min`](#fn-minmax) | qualquer | Mesma unidade que a entrada | Qualquer |
| [`percentage`](#fn-percentage) | `*` ou atributo, `WHERE` condição | % | Qualquer |
| [`percentile`](#fn-percentile) | numérico, nível `p` | Mesma unidade que a entrada | Qualquer |
| [`percentOfTotal`](#fn-percentoftotal) | numérico | % | Qualquer (não `TIMESERIES`) |
| [`rate`](#fn-rate) | contador numérico | Por segundo | Entidades contadoras (precisa de `TIMESERIES`) |
| [`stddev`](#fn-stddev) | numérico | Mesma unidade que a entrada | Qualquer |
| [`sum`](#fn-sum) | numérico | Mesma unidade que a entrada | Qualquer |
| [`uniqueCount`](#fn-uniquecount) | qualquer | Valores distintos | Qualquer |

<a id="fn-apdex"></a>

<details>
<summary><code>apdex(attribute, T)</code></summary>

Calcula uma pontuação de [Apdex](https://www.apdex.org/) de 0 (frustrado) a 1 (satisfeito), usando o limite `T` na unidade do atributo (milissegundos para `duration`).

| Balde | Condição | Peso |
| ---------- | ------------------------------------ | ------ |
| Satisfeito | `duration ≤ T` e não um erro | 1 |
| Tolerando | `T < duration ≤ 4T` e não um erro | 0.5 |
| Frustrado | `duration > 4T`, **ou qualquer erro** | 0 |

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service
```

- Funciona somente em entidades de rastreamento (`Transaction`, `AppRequest`, `Span`).
- `T` deve ser maior que 0. `FILTER` não é permitido.
- `apdex(duration, 500)` em `Transaction` é acelerado por rollup em janelas longas.

</details>

<a id="fn-average"></a>

<details>
<summary><code>average(attribute)</code> / <code>avg(attribute)</code></summary>

A média aritmética de um atributo numérico.

```sql
SELECT average(duration) AS avgMs FROM Transaction FACET service TIMESERIES AUTO
```

</details>

<a id="fn-count"></a>

<details>
<summary><code>count(*)</code> / <code>count(attribute)</code></summary>

- `count(*)` conta linhas.
- `count(attribute)` conta linhas nas quais o atributo é **não NULL**. Para atributos de cadeia de caracteres, os valores ausentes estão em branco (`''`) em vez de NULL, portanto, eles ainda são contados (consulte [Manuseio de tipo](#46-type-handling-blank-vs-null)).

```sql
SELECT count(*) AS requests, count(http.statusCode) AS withStatusCode FROM Transaction
```

</details>

<a id="fn-derivative"></a>

<details>
<summary><code>derivative(attribute)</code></summary>

A alteração por segundo de `average(attribute)` entre compartimentos consecutivos: (este compartimento - compartimento anterior) ÷ segundos do compartimento. O resultado pode ser negativo. Use-o em medidores para ver se um valor está crescendo ou diminuindo.

```sql
SELECT derivative(value) AS heapGrowthPerSec FROM JvmHeapUsed WHERE host = 'publish-1' TIMESERIES 5 minutes
```

- Requer `TIMESERIES`, e deve ser a agregação **only**. `FILTER` não é permitido.
- O primeiro bucket não tem antecessor.

</details>

<a id="fn-latest"></a>

<details>
<summary><code>latest(attribute [, rankAttribute])</code> / <code>earliest(…)</code></summary>

- `latest(attr)` retorna o valor da linha mais recente. `earliest(attr)` retorna o valor da linha mais antiga.
- Com um segundo argumento, a linha é escolhida por esse atributo em vez do tempo. `latest(traceId, duration)` retorna a ID de rastreamento da solicitação **mais lenta**. `earliest(traceId, duration)` retorna o mais rápido.

```sql
SELECT latest(value) AS current FROM MemoryUtilization FACET host
```

```sql
SELECT max(duration) AS slowestMs, latest(traceId, duration) AS slowestTrace FROM Transaction FACET name ORDER BY slowestMs DESC LIMIT 10
```

</details>

<a id="fn-minmax"></a>

<details>
<summary><code>max(attribute)</code> / <code>min(attribute)</code></summary>

O maior ou menor valor. Funciona com números, sequências e carimbos de data e hora.

```sql
SELECT min(duration) AS fastestMs, max(duration) AS slowestMs, max(timestamp) AS lastSeen FROM Transaction FACET service
```

</details>

<a id="fn-percentage"></a>

<details>
<summary><code>percentage(*, WHERE condition)</code></summary>

O compartilhamento de linhas que correspondem a `condition`, de 0 a 100.

```sql
SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction FACET service TIMESERIES AUTO
```

```sql
SELECT percentage(*, WHERE duration > 2000) AS slowPct FROM Transaction
```

`percentage` tem sua própria condição, portanto, não leva `FILTER`.

</details>

<a id="fn-percentile"></a>

<details>
<summary><code>percentile(attribute, p)</code></summary>

O percentual p aproximado, onde `0 < p ≤ 100`. Use um `percentile` por nível.

```sql
SELECT percentile(duration, 50) AS p50, percentile(duration, 95) AS p95, percentile(duration, 99) AS p99 FROM Transaction TIMESERIES AUTO
```

p50, p95 e p99 em `Transaction` são acelerados por rollup em janelas longas.

</details>

<a id="fn-percentoftotal"></a>

<details>
<summary><code>percentOfTotal(attribute)</code></summary>

Compartilhamento de cada linha de faceta da **soma total** de um atributo numérico. As linhas somam até 100%.

```sql
SELECT sum(duration) AS totalMs, percentOfTotal(duration) AS shareOfTime FROM Transaction FACET name LIMIT 15
```

Não permitido com `TIMESERIES`.

</details>

<a id="fn-rate"></a>

<details>
<summary><code>rate(attribute)</code></summary>

A taxa por segundo de um **contador cumulativo**. Em cada compartimento, `rate` obtém a maior leitura de contador por série e a diferencia do compartimento anterior. As reinicializações de contador são liberadas em 0, e o primeiro bucket é 0.

```sql
SELECT rate(value) AS bytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

- Funciona somente em entidades de contador (marcado ✔ no [Dicionário de dados](#53-curated-metric-entities)).
- Requer `TIMESERIES`, e deve ser a agregação **only**. `FILTER` não é permitido.

> [!WARNING]
> Sempre `FACET host`, ou filtrar para um único host. Um `rate()` não facetado em vários hosts compara contadores de hosts diferentes e produz números sem sentido. Os contadores de disco são por dispositivo, portanto, `FACET host, device` é o mais preciso.

</details>

<a id="fn-stddev"></a>

<details>
<summary><code>stddev(attribute)</code></summary>

O desvio padrão da amostra. Use-o com `average` para avaliar a consistência da latência.

```sql
SELECT average(duration) AS avgMs, stddev(duration) AS stddevMs FROM Transaction FACET name LIMIT 20
```

</details>

<a id="fn-sum"></a>

<details>
<summary><code>sum(attribute)</code></summary>

O total de um atributo numérico.

```sql
SELECT sum(callCount) AS calls FROM SpanBreakdown FACET segment LIMIT 20
```

</details>

<a id="fn-uniquecount"></a>

<details>
<summary><code>uniqueCount(attribute)</code></summary>

O número **exato** de valores distintos.

```sql
SELECT uniqueCount(host) AS hosts, uniqueCount(traceId) AS traces FROM Transaction FACET service
```

</details>

Acesso rápido às restrições de função:

| Restrição | Funções |
| ------------------------------------- | ----------------------------------------------------------------------------------------- |
| Atributo numérico obrigatório | `average`, `sum`, `percentile`, `stddev`, `percentOfTotal`, `rate`, `derivative`, `apdex` |
| Nenhum `FILTER (WHERE …)` | `apdex`, `rate`, `derivative`, `percentage` |
| Requer `TIMESERIES`, somente agregação | `rate`, `derivative` |
| Não com `TIMESERIES` | `percentOfTotal` |
| Rastrear somente entidades | `apdex` |
| Somente entidades contadoras | `rate` |

### Operadores e condições

| Operador | Significado | Exemplo |
| ------------------------- | -------------------------------------------------------------------------- | ---------------------------------------- |
| `=` | Igual | `status = 'Error'` |
| `!=`, `<>` | Diferente de | `kind != 'Internal'` |
| `<`, `<=`, `>`, `>=` | Comparação | `http.statusCode >= 500` |
| `LIKE` | Correspondência curinga: `%` = qualquer caractere, `_` = um caractere. Diferenciação de maiúsculas e minúsculas. | `urlPath LIKE '/content/%'` |
| `NOT LIKE` | Curinga negado | `urlPath NOT LIKE '%.css'` |
| `IN (…)` | Corresponde a qualquer um dos valores 1-100 | `service IN ('checkout', 'cart')` |
| `NOT IN (…)` | Não corresponde a nenhum dos valores | `http.method NOT IN ('OPTIONS', 'HEAD')` |
| `IS NULL` / `IS NOT NULL` | Verificação nula | `http.statusCode IS NOT NULL` |
| `AND`, `OR`, `NOT` | Lógica | `NOT (status = 'Ok')` |
| `( … )` | Agrupamento | `(a = 1 OR b = 2) AND c = 3` |

```sql
SELECT count(*) FROM Transaction WHERE (status = 'Error' OR duration > 2000) AND NOT host LIKE '%dispatcher%' AND http.method IN ('GET', 'POST')
```

> [!WARNING]
> `attr = NULL` é um erro. Usar `attr IS NULL`.

### Manuseio de tipo: em branco vs. NULL

Cada atributo tem um tipo no catálogo: `string`, `number`, `boolean` ou `timestamp`. As funções de verificação de tipo SQL em relação a ele (por exemplo, `average(service)` foi rejeitado). Nenhuma função de conversão do tipo é necessária ou está disponível.

> [!IMPORTANT]
> **Em branco vs. NULL.** A maioria dos atributos de cadeia de caracteres vem de mapas de atributos OpenTelemetry, onde uma chave _ausente_ é lida como uma cadeia de caracteres vazia `''`, não `NULL`. Para manter somente os spans que possuem um atributo, use **`attr != ''`** em vez de `IS NOT NULL`. Atributos numéricos como `http.statusCode` são genuinamente `NULL` quando ausentes, então `IS NOT NULL` funciona para eles.

```sql
SELECT count(*) AS jcrCalls FROM Span WHERE jcr.operation != '' FACET jcr.operation
```

### Traduzir SQL para SQL

| SQL | SQL |
| ------------------------------------------------------------ | ------------------------------------------------------------------- |
| `WHERE ts > now() - INTERVAL 1 HOUR` | Nada; o seletor de hora (ou `SINCE 1 hour ago`) define a janela |
| `GROUP BY service` | `FACET service` |
| `GROUP BY toStartOfInterval(ts, INTERVAL 5 MINUTE), service` | `FACET service TIMESERIES 5 minutes` |
| `quantile(0.95)(duration)` | `percentile(duration, 95)` |
| `countIf(status = 'Error')` | `count(*) FILTER (WHERE status = 'Error')` |
| `100 * countIf(x) / count()` | `percentage(*, WHERE x)` |
| `count(DISTINCT host)` | `uniqueCount(host)` |
| `argMax(traceId, duration)` | `latest(traceId, duration)` |
| `x / sum(x) OVER ()` | `percentOfTotal(x)` |
| `ORDER BY p95 DESC LIMIT 20` | `ORDER BY p95 DESC LIMIT 20` |
| `HAVING count() > 100` | _Sem suporte_. Classificar e `LIMIT` em vez disso. |
| `JOIN` / subconsulta | _Sem suporte_. Use um widget **Fórmula**. |
| `CASE WHEN` | _Sem suporte_. Use `FILTER` colunas ou `WHERE … LIKE` + `FACET`. |

### Importação de painéis existentes para Insights

Se você conhece a consulta existente, veja a seguir os detalhes da migração

| Consulta Existente | SQL | Notas |
| ------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------------------------ |
| `FROM Transaction SELECT count(*)` | Igual | Há suporte para `FROM`-primeira ordem |
| `SINCE 1 day ago` | Igual | Ignorado nos painéis; o seletor vence |
| `TIMESERIES AUTO` / `MAX` / `5 minutes` | Igual | `AUTO` andar é 2 minutos |
| `FACET appName` | `FACET appName` / `FACET service` |                                                              |
| `percentage(count(*), WHERE error IS true)` | `percentage(*, WHERE status = 'Error')` |                                                              |
| `filter(count(*), WHERE …)` | `count(*) FILTER (WHERE …)` | Formulário padrão SQL |
| `apdex(duration, t: 0.5)` | `apdex(duration, 500)` | `duration` está em **ms** |
| `latest(x)` / `earliest(x)` | Igual | Além de um formulário classificado: `latest(x, rankAttr)` |
| `uniqueCount(x)` | Igual | Exato |
| `rate(count(*), 1 minute)` | `count(*)` com `TIMESERIES 1 minute` | O SQL `rate()` é somente para contadores |
| `derivative(x, 1 minute)` | `derivative(x)` + `TIMESERIES` | Por segundo |
| `uniques(x)` | `count(*) … FACET x LIMIT n` |                                                              |
| `keyset()` / `SHOW EVENT TYPES` | [Consultas de descoberta](#101-data-discovery) ou o painel Guia de consulta |                                                              |
| `histogram(duration, …)` | `FILTER` compartimentos | Ver [Tutorial 3](#84-build-a-latency-histogram) |
| `FACET CASES (…)` | `FILTER` colunas | Ver [Tutorial 3](#83-group-values-into-cases-with-filter) |
| `COMPARE WITH 1 week ago` | Não suportado | Altere o seletor de hora ou use `SINCE … UNTIL …` por meio da API |
| `RLIKE` | `LIKE` / `IN` |                                                              |
| Matemática em `SELECT` / agregação aninhada | Widget de **Fórmula** |                                                              |
| `WITH TIMEZONE`, `EXTRAPOLATE`, `SLIDE BY` | Não suportado |                                                              |
