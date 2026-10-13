---
source-git-commit: d3cd77e213674587152d6d33efa65328f420a547
workflow-type: tm+mt
source-wordcount: '671'
ht-degree: 1%
---
# Introdução

**Adobe Observability Insights Query Language (SQL)** — consulte, gráfico e painel de telemetria nos Adobe Observability Insights.

|                |                                                                  |
| -------------- | ---------------------------------------------------------------- |
| **Aplica-se a** | Interface do usuário do Insights → **Explorar** e **Painéis** (painéis personalizados) |
| **Público-alvo** | Engenheiros, SREs e usuários avançados que conhecem SQL |

> [!TIP]
> Todo bloco de código marcado como `sql` é uma consulta SQL que você pode colar diretamente no **Explorar → SQL** ou nos **Painéis → Adicionar widget → SQL**.

## Introdução ao SQL: o idioma da telemetria {#get-started-with-sql-the-language-of-your-telemetry}

Insights A linguagem de consulta é uma linguagem de consulta semelhante a SQL para seus dados de Insights de observação. É o mesmo idioma atrás de cada widget de painel personalizado e da página **Explorar**, para que você possa criar os modos de exibição de que sua equipe precisa.

O SQL funciona com **entidades**, que são exibições lógicas amigáveis, como `Transaction`, `Span` ou `CpuUtilization`. Você nunca consulta tabelas de armazenamento diretamente. O Observability Insights compila cada consulta SQL em uma consulta de warehouse segura e com parâmetros. Ele sempre mapeia essa consulta para seu locatário e a janela de tempo selecionada.

```sql
SELECT count(*) AS requests FROM Transaction TIMESERIES AUTO
```

### O que você pode fazer com o SQL

Usar SQL para:

- **Criar gráficos** de taxa de transferência, taxa de erro, latência, Apdex, CPU, memória, heap de JVM, disco e rede ao longo do tempo.
- **Responda a perguntas específicas**, como &quot;quais caminhos de URL no nível de publicação têm o pior p95?&quot; ou &quot;which downstream domain is failed?&quot; (que domínio downstream está falhando?)
- **Investigue incidentes** aprofundando de um KPI em solicitações individuais e IDs de rastreamento.
- **Crie painéis** com KPIs, tendências, tabelas, fórmulas, notas, cabeçalhos de seção e variáveis.
- **Criar perfil das cargas de trabalho do AEM** (Adobe Experience Manager) usando atributos Sling, JSP, JCR, dispatcher e GraphQL.

O SQL dá suporte a **consultas de eventos**, que retornam linhas de solicitação individuais, e **consultas agregadas**, que retornam totais, percentis, taxas, detalhamentos e séries de tempo.

> [!TIP]
> Os modelos em **Painéis → Criar painel** são criados inteiramente do SQL. Abra qualquer widget de modelo com o ícone de lápis para ver e aprender com sua consulta.

### O que você pode consultar

O SQL pode consultar quatro famílias de dados. A lista completa está no [Dicionário de dados](#data-dictionary).

| Família | Entidades | Perguntas típicas |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| **Rastreamentos (APM)** | `Transaction` (solicitações de entrada), `AppRequest` (solicitações + consumidores de mensagem), `Span` (cada intervalo) | Taxa de transferência, latência, erros, Apdex, chamadas downstream, caminhos de código do AEM |
| **Métricas de host e tempo de execução preparadas** | `CpuUtilization`, `MemoryUtilization`, `LoadAverage5m`, `DiskUtilization`, `NetworkReceiveRate`, `JvmHeapUsed`, `JvmThreadCount`, `DispatcherRequestRate` e mais | Saturação, capacidade, integridade do JVM, carga do dispatcher |
| **Métricas genéricas** | `MetricGauge`, `MetricSum` | Qualquer métrica OpenTelemetry por nome, descoberta de métrica |
| **Composição de extensão pré-agregada** | `SpanBreakdown` | Quais operações secundárias compõem uma transação |

> [!NOTE]
> Logs, definições de alerta e relações de inventário não estão disponíveis.

### Onde usar SQL

| Onde | Como abri-lo | Para que serve |
| ----------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Explorar** | Navegação à esquerda → **Explorar** | Ad hoc analysis Use o **Visual Builder** ou a guia **SQL**, **Validar** e **Executar** (⌘/Ctrl + Enter). Inclui consultas **Histórico** e **Salvas** e um painel **Guia de consulta**. |
| **Painéis** | Navegação à esquerda → **Painéis** → abrir ou criar um painel → **Adicionar widget** | Gráficos persistentes, KPIs e tabelas, combinados em painéis. |
| **Widgets de fórmula** | **Adicionar widget → Fórmula** | Combine 2 a 6 queries SQL com aritmética (taxas, espaço livre, taxas de gravação). |
| **Variáveis do painel** | **+ Variável → De uma consulta** | Preencha um seletor dos valores distintos de uma consulta SQL `FACET`. |
| **Inspeção de consulta** | Passe o mouse sobre um gráfico → inspecionar consulta | Veja o SQL por trás de um gráfico incorporado ou de painel. |

### Estrutura de consulta SQL

O SQL segue uma sintaxe baseada em cláusula. Essa consulta conta todas as solicitações de entrada na janela selecionada:

```sql
SELECT count(*) FROM Transaction
```

Estas são todas as cláusulas disponíveis:

```text
SELECT  function(attribute) [FILTER (WHERE condition)] [AS alias] [, ...]
        | attribute [AS alias] [, ...]
FROM    Entity
  [WHERE      condition [AND | OR ...]]
  [FACET      attribute [, attribute ...]]        -- up to 5
  [SINCE      n unit [AGO]]
  [UNTIL      NOW | n unit [AGO]]
  [TIMESERIES AUTO | MAX | n unit]
  [ORDER BY   alias [ASC | DESC]]
  [LIMIT      number]                              -- 1 to 5000
```

Estas são as regras que o SQL segue:

| Regra | Detalhes |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Cláusulas obrigatórias** | `SELECT` e `FROM` são obrigatórios. Qualquer outra cláusula é opcional. Você pode começar com: `FROM Transaction SELECT count(*)` também é válido. |
| **Ordem da cláusula** | As cláusulas depois de `FROM` podem aparecer em qualquer ordem. Cada cláusula pode aparecer apenas uma vez. |
| **Tamanho da consulta** | Uma consulta deve ter 4 KB ou menos. |
| **Diferenciação de maiúsculas e minúsculas** | Palavras-chave, nomes de funções, nomes de entidades e nomes de atributos diferenciam maiúsculas de minúsculas **não**. Os valores da cadeia de caracteres **diferenciam maiúsculas de minúsculas** (`'Error'` ≠ `'error'`). |
| **Cadeias de caracteres** | Usar aspas simples: `WHERE status = 'Error'`. Para incluir uma aspa, dobre-a: `'O''Brien'`. Aspas duplas não são aceitas. |
| **Literais** | Sequências, números (incluindo negativos e decimais), `TRUE`, `FALSE`, `NULL`. |
| **Nomes de atributos** | Os nomes podem conter letras, dígitos, `_` e `.` (por exemplo, `http.statusCode`). Não é necessário usar backticks. |
| **Agregações vs. atributos** | Uma consulta seleciona atributos brutos (uma lista de eventos) ou agregações, nunca ambos. Para agrupar, use `FACET`. |
| **Janela de tempo** | O padrão é a última hora. O máximo é de 31 dias. Nos painéis e em Explorar, o **seletor de horas** define a janela (consulte [Janela de horas](#time-window-precedence)). |
| **Sem suporte** | Ponto e vírgula, comentários, subconsultas, junções, `SELECT *`, matemática entre atributos em `SELECT`, `HAVING`, `COMPARE WITH`. Consulte [O que o SQL não faz](#what-sql-doesnt-do) para ver as alternativas. |
