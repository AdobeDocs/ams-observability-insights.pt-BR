---
source-git-commit: ec98f6bd132cadaba1da7c4fb727d7e031790645
workflow-type: tm+mt
source-wordcount: '327'
ht-degree: 5%
---
## Resolução de problemas {#troubleshooting}

Mensagens de erro aparecem no editor quando você **Valida** ou **Executa** uma consulta e no próprio widget em um painel. Localize a mensagem na primeira coluna.

| Mensagem/sintoma | Causa | Correção |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| `Unknown entity 'X'` | Erro de digitação ou nome de não-catálogo | Verifique o [Dicionário de dados](#data-dictionary) / o painel Guia de consulta. |
| `Attribute 'x' is not available on Entity` | O atributo não está nessa entidade (ex.: `duration` em `CpuUtilization`) | Atributos de rastreamento vs. atributos de métrica — consulte [Dicionário de dados](#data-dictionary). |
| `Raw attributes cannot be mixed with aggregates; use FACET to group results` | `SELECT service, count(*)` | `SELECT count(*) … FACET service`. |
| `FACET and TIMESERIES require an aggregate query` | Consulta de evento com FACET/TIMESERIES | Use uma agregação. |
| `FACET supports at most five attributes` | > 5 facetas | Reduzir. |
| `average requires a numeric attribute` | e.g. `average(service)` | Use `uniqueCount`/`latest` ou um atributo numérico. |
| `rate requires TIMESERIES` / `rate is only available for counter measures` | `rate()` sem SÉRIE TEMPORAL ou em um medidor | Adicionar `TIMESERIES AUTO`; usar uma entidade de contador ✔ ([Entidades de métrica preparadas](#53-curated-metric-entities)); para medidores, usar `derivative()`. |
| `rate cannot be combined with other aggregates` | Vários itens com `rate()` | Um `rate()` por widget; combinar com uma Fórmula. |
| `percentOfTotal cannot be combined with TIMESERIES` | — | Remover SÉRIE TEMPORAL. |
| `apdex requires an entity with a status attribute` | `apdex` em uma entidade de métrica | Use `Transaction`/`Span`/`AppRequest`. |
| `ORDER BY 'x' must reference a selected value or alias` | Ordenação por algo não selecionado | Alias: a agregação e a ordem por alias. |
| `IN requires between 1 and 100 values` | — | Dividir ou usar `LIKE`. |
| `Use IS NULL or IS NOT NULL for null comparisons` | `attr = NULL` | Usar `IS NULL`. |
| `Unexpected character '{'` | `{{variable}}` na pré-visualização do widget Adicionar ou ao salvar | Consulte o [Tutorial 3, etapa 8.7](#87-make-dashboards-reusable-with-variables) - Limitação. |
| `Unexpected character ';'` / `'"'` | Ponto e vírgula ou aspas duplas | Remover `;`; usar aspas simples. |
| `Expected a positive duration such as 30 minutes` | `TIMESERIES 5m` | Gravar `TIMESERIES 5 minutes`. |
| `Time range cannot exceed 31 days` | Seletor > 31 d | Restrinja a janela. |
| `Select a customer before running a SQL query` | Nenhum locatário / locatário de demonstração | Escolha um cliente no alternador de locatário. |
| `All queries in a formula must be the same kind…` | Combinação de total e SÉRIES TEMPORAIS ou uma FACETA | Tornar todas as subconsultas simples ou todas `TIMESERIES AUTO`; remover FACET. |
| `Query 'x' must select exactly one metric…` | A subconsulta tem duas agregações | Uma agregação por subconsulta. |
| `Unknown query name 'x' - must be one of: …` | A fórmula faz referência a um nome ausente | Corresponder nomes exatamente (diferencia maiúsculas de minúsculas). |
| **Falha ao salvar** | Uma consulta de widget é inválida no lado do servidor (frequentemente `{{var}}`) | Abra cada widget editado recentemente e **execute**; corrija ou remova o widget inválido. |
| _Selecione um valor para{{x}}_ | A variável não tem seleção | Escolha um valor na barra de variáveis. |
| _Nenhuma atividade nos últimos ..._ | Nenhuma linha na janela | Amplie a janela; verifique os filtros com uma consulta de descoberta ([Descoberta de dados](#101-data-discovery)). |
| Linhas quebradas em pontos | Compartimento menor que a cadência de dados | Use `TIMESERIES AUTO` (≥ 2 min) ou um gráfico de barras. |
| Faceta mostra um grupo de sequência vazia | Atributo ausente em algumas linhas | Adicionar `WHERE attr != ''`. |
| _Este painel SQL herdado não é mais executado_ | Painel do construtor Raw-SQL antigo | Editar → reconstruir no SQL. |
