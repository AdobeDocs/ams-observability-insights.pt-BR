---
source-git-commit: 263ba9e9b3eb1c6e4dd253ca03bd0ce8954c588b
workflow-type: tm+mt
source-wordcount: '562'
ht-degree: 0%
---
## Tutorial 1: Suas primeiras consultas {#tutorial-1-your-first-queries}

Bem-vindo ao SQL. Neste tutorial, você escreverá suas primeiras consultas, controlará a janela de tempo, plotará tendências, filtrará e agrupará resultados e colocará seu primeiro gráfico em um painel. Cada etapa se baseia na anterior.

**Neste tutorial:** [Antes de começar](#before-you-begin) · [6.1 Eventos de consulta](#61-query-events-with-select-from-and-limit) · [6.2 Agregação](#62-aggregate-your-data) · [6.3 Várias agregações](#63-return-several-aggregates-at-once) · [6.4 Janela de tempo](#64-set-the-time-window) · [6.5 SÉRIES TEMPORAIS](#65-plot-trends-with-timeseries) · [6.6 ONDE](#66-filter-with-where) · [6.7 FACET](#67-group-with-facet) · [6.8 Salvar em um painel](#68-save-your-query-to-a-dashboard)

### Antes de começar

- Você precisa de acesso aos Insights de observação e, pelo menos, um locatário do cliente que envia rastreamentos ou métricas.
- Selecione esse **cliente** no alternador de locatário. O SQL não é executado enquanto **Todas as contas** estiverem selecionadas.
- Abra **Explorar** e alterne para a guia **SQL**. Use **Executar** (⌘/Ctrl + Enter) após cada consulta.

### Consultar eventos com `SELECT`, `FROM` e `LIMIT`

Toda consulta precisa de duas cláusulas: `SELECT` diz o que retornar e `FROM` diz qual entidade ler. Iniciar com solicitações de entrada:

```sql
SELECT timestamp, service, name, duration FROM Transaction
```

Esta é uma **consulta de evento**. Cada linha é uma solicitação. Sem um `LIMIT`, você obtém até 100 linhas. Solicitar menos:

```sql
SELECT timestamp, service, name, duration FROM Transaction LIMIT 10
```

As linhas são classificadas pela primeira coluna, em ordem decrescente, portanto, colocar `timestamp` primeiro fornece as solicitações mais recentes na parte superior.

> [!NOTE]
> O SQL não tem `SELECT *`. Os atributos de listagem mantêm os resultados pequenos e legíveis. O [Dicionário de dados](#data-dictionary) e o preenchimento automático no editor mostram o que está disponível.

### Agregar seus dados

A maioria dos painéis não mostra eventos individuais. Eles as resumem. Substitua os atributos por uma **função agregadora**:

```sql
SELECT count(*) FROM Transaction
```

Esse é o número de solicitações de entrada na janela. Tente mais alguns:

```sql
SELECT average(duration) FROM Transaction
```

```sql
SELECT max(duration) FROM Transaction
```

```sql
SELECT uniqueCount(host) FROM Transaction
```

### Retornar várias agregações de uma vez

Separe as agregações com vírgulas e nomeie cada uma com `AS`:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs, max(duration) AS maxMs FROM Transaction
```

> [!TIP]
> Os aliases se tornam cabeçalhos de coluna e rótulos de legenda, e são o que você menciona em `ORDER BY`. Aprenda a nomear cada agregado.

### Definir a janela de tempo

Por padrão, uma consulta abrange a **última hora**. Use `SINCE` para retroceder:

```sql
SELECT count(*) AS requests FROM Transaction SINCE 1 day ago
```

Use `UNTIL` para encerrar a janela no passado. Isso conta o tráfego de ontem, de 48 a 24 horas atrás:

```sql
SELECT count(*) AS requests FROM Transaction SINCE 2 days ago UNTIL 1 day ago
```

As unidades podem ser `second`, `minute`, `hour`, `day` ou `week`. A janela pode ter no máximo 31 dias.

> [!IMPORTANT]
> No **Explorar** e nos **painéis**, o **seletor de horas** na parte superior da página controla a janela e substitui `SINCE`/`UNTIL`. Para observar outro ponto, altere o seletor. O restante deste guia deixa `SINCE` fora das consultas do painel por esse motivo.

### Plotar tendências com `TIMESERIES`

Adicione `TIMESERIES` para transformar um único número em uma linha ao longo do tempo:

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

`AUTO` escolhe um tamanho de balde para que o gráfico tenha cerca de 120 pontos (2 minutos para uma janela de 1 hora, 15 minutos para 24 horas). Também é possível definir o bucket explicitamente:

```sql
SELECT average(duration) AS avgMs FROM Transaction TIMESERIES 5 minutes
```

### Filtrar com `WHERE`

`WHERE` mantém apenas as linhas importantes para você. Contar somente solicitações com falha:

```sql
SELECT count(*) AS errors FROM Transaction WHERE status = 'Error'
```

Combinar condições com `AND`, `OR` e parênteses:

```sql
SELECT count(*) AS slowOrFailed FROM Transaction WHERE service = 'checkout' AND (status = 'Error' OR duration > 2000)
```

Usar `LIKE` com `%` para correspondências parciais:

```sql
SELECT count(*) AS contentRequests FROM Transaction WHERE urlPath LIKE '/content/%'
```

> [!NOTE]
> Os valores da cadeia de caracteres diferenciam maiúsculas de minúsculas e usam aspas simples: `'Error'`, não `"Error"` ou `'error'`. Substitua `checkout` com um serviço de seu locatário. A próxima etapa mostra como listá-los.

### Agrupar com `FACET`

`FACET` resultados de grupos, como `GROUP BY` de SQL. Liste seus serviços e o grau de ocupação de cada um:

```sql
SELECT count(*) AS requests FROM Transaction FACET service
```

Uma consulta facetada retorna os 10 grupos principais por padrão. Use `LIMIT` para ver mais:

```sql
SELECT count(*) AS requests, average(duration) AS avgMs FROM Transaction FACET service LIMIT 50
```

Combine `FACET` e `TIMESERIES` para obter uma linha por grupo:

```sql
SELECT count(*) AS requests FROM Transaction FACET service TIMESERIES AUTO LIMIT 5
```

Aqui, `LIMIT 5` mantém as cinco séries mais ocupadas.

### Salvar a consulta em um painel

1. Abra **Painéis → Criar painel → Criar um painel personalizado** e nomeie-o _Meu primeiro painel_.
2. Clique em **Adicionar widget**, escolha **Consulta única** e alterne para **SQL**.
3. Cole a consulta da etapa 6.7 e clique em **Executar consulta**.
4. Defina o **Título** como _Solicitações por serviço_ e escolha o tipo de gráfico **Linha**.
5. Clique em **Salvar widget** e depois em **Salvar** na barra superior.

Você criou seu primeiro painel.
