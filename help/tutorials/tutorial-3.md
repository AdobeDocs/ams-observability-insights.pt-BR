---
title: 'Tutorial 3: Avançar seu painel'
description: Crie painéis com variáveis, fórmulas e vários widgets no Observability Insights.
source-git-commit: 91a61d33010192cedce7d40a45ffe86f32e4824b
workflow-type: tm+mt
source-wordcount: '879'
ht-degree: 3%
---

## Tutorial 3: Avançar seu painel {#tutorial-3-advance-your-dashboarding}

Neste tutorial, você criará os componentes de um painel de nível de produção: pontuações de experiência do usuário, taxas de contador, agrupamento de casos, histogramas, blocos de KPI com limites, seções, widgets de fórmulas e variáveis.

### Pontuar experiência do usuário com `apdex`

O Apdex transforma latência e erros em uma pontuação de 0 a 1. Com um limite de 500 ms, uma solicitação é _atendida_ em ou abaixo de 500 ms, _tolerante_ até 2.000 ms e _frustrada_ acima desse **ou se falhar**:

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction
```

Analisar a tendência por serviço:

```sql
SELECT apdex(duration, 500) AS apdex FROM Transaction FACET service TIMESERIES AUTO LIMIT 8
```

Coloque-o em um widget **Número** com limites `0 → Red`, `0.7 → Amber`, `0.85 → Green`. A banda mais alta vence, então as notas baixas ficam vermelhas.

### Transformar contadores em taxas com `rate`

As entidades contadoras (✔ no [Dicionário de dados](#53-curated-metric-entities)) armazenam totais cada vez maiores. `rate(value)` os converte em valores por segundo. Ele precisa de `TIMESERIES`, e deve ser a única agregação:

```sql
SELECT rate(value) AS rxBytesPerSec FROM NetworkReceiveRate FACET host TIMESERIES AUTO
```

```sql
SELECT rate(value) AS writeBytesPerSec FROM DiskWriteRate FACET host, device TIMESERIES AUTO LIMIT 10
```

> **Aviso**
>
> Sempre encare `rate()` por `host` (e por `device` para discos) ou filtre para um host. Sem isso, as leituras de diferentes hosts se misturam e o resultado é sem sentido.

Para obter uma taxa de **solicitação** dos rastreamentos, conte as solicitações em compartimentos fixos. Isso fornece solicitações por minuto:

```sql
SELECT count(*) AS requestsPerMinute FROM Transaction TIMESERIES 1 minute
```

### Agrupar valores em ocorrências com `FILTER`

`FACET CASES` do NRQL agrupa linhas em compartimentos nomeados. No SQL, grave uma coluna `FILTER` por caso. Estas são as classes de status por serviço:

```sql
SELECT count(*) FILTER (WHERE http.statusCode < 400) AS ok, count(*) FILTER (WHERE http.statusCode >= 400 AND http.statusCode < 500) AS clientErrors, count(*) FILTER (WHERE http.statusCode >= 500) AS serverErrors FROM Transaction FACET service LIMIT 20
```

A mesma ideia funciona com o tempo, fornecendo uma linha por caso:

```sql
SELECT count(*) FILTER (WHERE urlPath LIKE '/content/%') AS pages, count(*) FILTER (WHERE urlPath LIKE '/bin/%') AS servlets, count(*) FILTER (WHERE urlPath LIKE '/graphql/%') AS graphql FROM Transaction TIMESERIES AUTO
```

### Criar um histograma de latência

O SQL não tem função `histogram()`. Compile os compartimentos com `FILTER` e renderize o resultado como um widget de **Barra** ou **Tabela**:

```sql
SELECT count(*) FILTER (WHERE duration <= 100) AS 'le 100ms', count(*) FILTER (WHERE duration > 100 AND duration <= 250) AS '100-250ms', count(*) FILTER (WHERE duration > 250 AND duration <= 500) AS '250-500ms', count(*) FILTER (WHERE duration > 500 AND duration <= 1000) AS '0.5-1s', count(*) FILTER (WHERE duration > 1000 AND duration <= 2000) AS '1-2s', count(*) FILTER (WHERE duration > 2000) AS 'over 2s' FROM Transaction
```

Adicione `FACET service` para comparar as distribuições de vários serviços lado a lado.

### Criar blocos e seções de KPI

Um bom painel começa com uma linha de blocos de KPI e agrupa gráficos de detalhes em seções.

1. Adicione quatro widgets **Number**, com largura de 3 colunas cada:

   | Título | Consulta | Unidade | Limites |
   | -------- | --------------------------------------------------------------------------- | ---- | ---------------------------------------- |
   | Solicitações | `SELECT count(*) AS requests FROM Transaction` |      |                                          |
   | Erro % | `SELECT percentage(*, WHERE status = 'Error') AS errorPct FROM Transaction` | `%` | `0 → Green`, `1 → Amber`, `5 → Red` |
   | p95 | `SELECT percentile(duration, 95) AS p95 FROM Transaction` | `ms` | `0 → Green`, `800 → Amber`, `2000 → Red` |
   | Apdex | `SELECT apdex(duration, 500) AS apdex FROM Transaction` |      | `0 → Red`, `0.7 → Amber`, `0.85 → Green` |

2. Adicione um widget **Texto**, marque _Transformar em cabeçalho de seção_ e chame-o de **LATÊNCIA**. Adicione gráficos de latência abaixo dele.
3. Adicione outro cabeçalho de seção, **ERROS**, seguido de gráficos de erros. Recolher uma seção com sua divisa para focalizar na outra.
4. Use **guias** para públicos diferentes, por exemplo _Visão geral_, _AEM_, _Infraestrutura_. Somente a guia visível executa suas consultas.

### Combinar consultas com widgets de fórmula

O SQL não pode fazer matemática em `SELECT` e não tem subconsultas. **Widgets de fórmula** preenchem essa lacuna: executam de 2 a 6 consultas nomeadas e combinam seus resultados com `+ - * / ( )`.

**Exemplo: solicitações por host ao longo do tempo** (Fórmula, Linha)

1. **Adicionar widget → Fórmula**.
2. Nomeie a primeira consulta `requests` e a segunda `hosts`.
3. Insira as consultas e a fórmula e clique em **Executar fórmula**.

Consulta `requests`:

```sql
SELECT count(*) AS v FROM Transaction TIMESERIES AUTO
```

Consulta `hosts`:

```sql
SELECT uniqueCount(host) AS v FROM Transaction TIMESERIES AUTO
```

Fórmula:

```text
requests / hosts
```

**Exemplo: compartilhamento de tempo gasto no banco de dados** (Fórmula, Número, unidade `%`)

Consulta `db`:

```sql
SELECT sum(duration) AS v FROM Span WHERE db.system != ''
```

Consulta `server`:

```sql
SELECT sum(duration) AS v FROM Transaction
```

Fórmula:

```text
(db / server) * 100
```

Regras para consultas de fórmula:

| Regra | Detalhes |
| --------------- | -------------------------------------------------------------------------------------------------------------------- |
| Nomes | Letras, dígitos e `_`, começando com uma letra ou `_`; até 40 caracteres; único |
| Expressão | `+ - * / ( )`, números e nomes de consulta; até 500 caracteres |
| Mesma forma | Todas as consultas são totais simples (Número), **ou** todas são `TIMESERIES` com o **mesmo tamanho de compartimento** (Linha/Área/Barra) |
| Uma métrica cada | Cada consulta retorna exatamente uma agregação e nenhuma `FACET` |
| Dados ausentes | Um valor ausente ou uma divisão por zero produz um ponto vazio, nunca `Infinity` ou `NaN` |

>[!TIP]
>
> Use `TIMESERIES AUTO` em cada consulta de uma fórmula de série temporal. Como os queries compartilham uma janela, eles sempre são resolvidos para o mesmo tamanho de bucket.

### Torne os painéis reutilizáveis com variáveis

As variáveis permitem que um painel sirva a muitos serviços, camadas ou limites. Cada variável é um seletor na barra de variáveis. Referencie-o como `{{name}}` em títulos de widget e widgets de texto (e, uma vez corrigida a limitação abaixo, em SQL de widget).

**Crie uma variável** (_org e super administradores_): clique em **+ Variável**.

| Texto | Notas |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Nome** | Letras, dígitos e `_`, começando com uma letra ou `_`. Deve ser única. Usado como `{{name}}`. |
| **Rótulo** | O que o seletor mostra (opcional). |
| **Tipo** | **Lista personalizada** (um valor por linha), **De uma consulta** (uma consulta SQL com um `FACET`; seus valores distintos se tornam opções) ou **Caixa de texto** (texto livre). |
| **Permitir seleção de vários valores** | Expande para uma lista separada por vírgulas. Use com `IN (…)`. |
| **Os valores são números** | Injeta valores sem aspas (`500` em vez de `'500'`). |

**Como funciona a substituição**

| Seleção | `{{svc}}` torna-se |
| ---------------------------------- | ------------------------------------------------------- |
| Cadeia de caracteres única `checkout` | `'checkout'` |
| Valores múltiplos `checkout`, `cart` | `'checkout', 'cart'` |
| Número `500` | `500` |
| Valor contendo `'` | Escapado como `''` |
| Nada selecionado | O widget aguarda e mostra _Selecionar um valor para{{svc}}_ |

```text
WHERE service = {{svc}}            single select
WHERE service IN ({{svc}})         multiple select
WHERE duration > {{slowMs}}        numeric text box
```

Para uma variável orientada por consulta, **sempre adicione um`LIMIT`**. Uma consulta facetada retorna somente os 10 valores principais por padrão. Isso lista até 200 serviços:

```sql
SELECT count(*) FROM Transaction FACET service LIMIT 200
```

As opções são atualizadas quando o locatário ou o seletor de hora é alterado e são classificadas em ordem alfabética. Quando a seleção atual não é mais válida, o primeiro valor (seleção única) ou todos os valores (seleção múltipla) são selecionados automaticamente. A consulta de uma variável não pode fazer referência a outras variáveis.

> **Aviso**
>
> **Limitação de compilação atual.** Ocorre substituição na grade do painel, mas a visualização do **Adicionar widget** e a **verificação de salvamento do lado do servidor** recebem o token `{{name}}` bruto e o rejeitam com `Unexpected character '{'`. Na prática:
>
> - `{{variables}}` em **títulos de widget** e **widgets de texto** funcionam de ponta a ponta.
> - `{{variables}}` no **SQL do widget** é renderizado corretamente após uma **Importação**, mas o painel não pode ser **salvo** enquanto esses widgets existirem.
>
> Até que isso seja corrigido, crie widgets por contexto com filtros literais (por exemplo `WHERE service = 'checkout'`) e use **Duplicar** para cloná-los.

### Coloque-o junto

Agora você tem todos os blocos de construção usados nos [blueprints do painel](../examples/dashboard-blueprint.md): blocos de KPI com limites, seções, percentis, Apdex, taxas, colunas de caso, histogramas, fórmulas e variáveis. Os blueprints mostram como combiná-los em painéis completos de sinais de ouro, triagem de erros, AEM, infraestrutura e JVM.
