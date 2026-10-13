---
title: Criar um painel
description: Saiba como criar, configurar, salvar e compartilhar painéis de Insights de observação.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '1129'
ht-degree: 1%
---

## Criar um painel, passo a passo

### Etapa 1: antes de começar

- **Selecione um único cliente** no alternador de locatário. **Criar painel** está desabilitado enquanto **Todas as contas** está selecionado. O SQL não pode ser executado no locatário de demonstração.
- **Funções.** Qualquer pessoa com acesso ao painel pode adicionar, editar e salvar widgets. Criar ou editar **variáveis** e inspecionar a consulta de um widget é limitado a **administradores da organização** e **superadministradores**.

### Etapa 2: criar o painel

1. Abra **Painéis** na navegação à esquerda.
2. Clique em **Criar painel**.
3. Escolha um ponto inicial:

   | Opção | O que você ganha |
   | ---------------------------- | -------------------------------------------------------------------------------------------------------- |
   | **Criar um painel personalizado** | Um painel vazio com uma guia. |
   | **Noções básicas sobre o CPU** | Host e processo CPU (usuário/sistema/iowait/roubo), throughput, carga, Apdex. |
   | **Visão geral do All-in-One** | KPIs (volume, % de erro, Apdex, p95), pilhas de erro/tráfego, tabelas de memória/CPU, threads de JVM, operações lentas. |
   | **Principais Transações** | Apdex, p95 por operação, carga por host/CPU, transações mais lentas com IDs de rastreamento, detalhamento de erros HTTP. |

4. Insira um **nome** e confirme. A tela do painel é aberta.

### Etapa 3: Conheça a tela

| Área | Controles |
| ---------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Barra superior** | **Adicionar widget**, **Guia** (folha de características no aplicativo), **Importar**, **Exportar**, **Salvar**, seletor de tempo e atualizar |
| **Barra de variáveis** | Selo de locatário, um seletor por variável, **+ Variável** (administradores) |
| **Barra de guias** | Alternar guias; renomear (lápis), excluir (×), adicionar (+) |
| **Grade** | 12 colunas Arraste os widgets pela alça, redimensione a partir do canto (mínimo 2×2). |
| **Cabeçalho do widget** (ao passar o mouse) | Baixar CSV, Baixar PNG, Editar, Duplicar, Remover, arrastar alça |

>[!IMPORTANT]
>
> **Salvar é explícito.** Alterações de layout, novos widgets, guias e variáveis permanecem no navegador até que você clique em **Salvar**. O botão mostra _Salvando... → Salvo_, ou _Falha ao salvar_ (consulte [Solução de problemas](#troubleshooting)).

### Etapa 4: adicionar um widget

Clique em **Adicionar widget**. A caixa de diálogo tem três modos: **Consulta única**, **Fórmula** e **Texto**. Ele também tem seu próprio seletor de tempo, para que você possa visualizar o widget em qualquer janela.

**Opção A: o Construtor (sem código)**

1. _(Opcional)_ **Comece com um auxiliar**: escolha uma das 57 predefinições, agrupadas como sinais dourados, APM, AEM, Transações, Rastreamentos e extensões, Infraestrutura, Editores, JVM e tempo de execução e Descoberta de dados. Uma predefinição preenche o título, a consulta, o tipo de gráfico e a unidade.
2. **Mostrar**: escolha `Count`, `Average`, `Percentile` (e seu nível), `Sum`, `Min`, `Max` ou `Unique count` e, em seguida, um atributo.
3. **de**: escolha a entidade.
4. **Filtro**: adicionar linhas de `attribute operator value` (`=`, `!=`, `>`, `<`, `>=`, `<=`, `LIKE`). Todas as linhas são combinadas com `AND`.
5. **Agrupar por**: escolha um atributo de faceta e **N principais** para manter apenas a série mais movimentada.
6. **Ao longo do tempo**: ativa o `TIMESERIES`, com um **a cada N minutos** tamanho de bucket.
7. Verifique o SQL gerado no formulário e clique em **Executar consulta**.

**Opção B: o editor SQL**

1. Alterne o **Construtor** para **SQL**. A edição de um widget existente sempre é aberta no SQL.
2. Digite ou cole uma consulta. O editor é preenchido automaticamente no catálogo em tempo real.
3. Use o painel **Guia de consulta** para procurar entidades, atributos e exemplos.
4. Clique em **Executar consulta**. Você verá que a consulta _`<Entity>`é válida_, uma visualização ao vivo e um resumo (`N rows · N time points · N groups`). Um aviso de _Série esparsa_ é exibido quando há menos de três pontos de tempo.

>[!TIP]
>
> O Construtor aborda as consultas mais comuns. Alternar para SQL para `apdex`, `percentage`, `rate`, `FILTER`, várias agregações, `OR`, `IN`, várias facetas ou `ORDER BY`.

### Etapa 5: configurar a aparência

| Texto | Notas |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Título** | Até 200 caracteres. Pode conter `{{variable}}` tokens. |
| **Tipo de gráfico** | `Line`, `Area`, `Bar`, `Pie / Donut`, `Number`, `Table`. Um tipo de gráfico é recomendado após cada execução até que você escolha um você mesmo. |
| **Unidade** | Opcional; até 40 caracteres (`ms`, `%`, `req/s`, `MB`, `bytes`). Deixe em branco para utilizar a unidade inferida (por exemplo `ms` para `duration`, `%` para `CpuUtilization`). |
| **Limites de cores** | Somente widgets de número. Até 10 bandas da forma _em ou acima `value` → Verde / Âmbar / Vermelho_. A banda mais bem sucedida ganha. |

### Etapa 6: salvar o widget e, em seguida, o painel

**O widget Salvar** só está habilitado depois que uma execução retorna dados. Os novos widgets são 6 colunas × 3 linhas e são empacotados dois por linha. Organize-os e clique em **Salvar** na barra superior.

## Escolha o gráfico correto

| Forma de consulta | Melhor gráfico | Por que |
| ------------------------------------------------- | ------------------- | ---------------------------------------------- |
| Uma agregação, não `FACET`, não `TIMESERIES` | **Número** | Um único bloco de KPI; adicione limites. |
| Agregado + `TIMESERIES` | **Linha** / **Área** | Uma tendência; uma série por valor de faceta. |
| Agregado + `FACET` (poucos valores) | **Barra** / **Pizza** | Compara categorias; pizza por compartilhamento de um todo. |
| Várias agregações + `FACET` | **Tabela** | Cada agregação se torna uma coluna. |
| Atributos brutos | **Tabela** | Uma lista de eventos. |
| Agregar + `FACET` + `TIMESERIES` em dados esparsos | **Barra** | Lê melhor do que linhas fracas. |

Quando a linha única de um widget Número carrega um valor de texto (por exemplo, `latest(host)`), o widget o mostra como um subtítulo.

## Widgets de texto: notas e cabeçalhos de seção

**Adicionar widget → Texto**:

- **Observação**: texto simples (até 4.000 caracteres, linhas em branco preservadas). Use-o para links runbook, proprietários e definições de SLO. `{{variables}}` são renderizados.
- **Cabeçalho da seção**: marque _Transformar em cabeçalho de seção_. Ela expande a largura total e sua divisa recolhe todos os widgets abaixo dela até o próximo cabeçalho da seção. Use-o para agrupamentos como **PUBLISH**, **AUTHOR** e **DISPATCHER**.

>[!TIP]
>
> Os títulos de widget que terminam em `- PUBLISH` ou `- AUTHOR` obtêm uma dica de cor sutil, o que facilita a verificação dos painéis do AEM por camada.

## O seletor de hora

- Ele define a janela para o widget **a cada** e para as opções de variáveis orientadas por consulta.
- Ele substitui `SINCE` / `UNTIL` em consultas de widget.
- A janela máxima é de **31 dias**.
- `TIMESERIES AUTO` adapta o tamanho do seu bucket à janela.
- O subtítulo de cada widget mostra a janela realmente aplicada, por exemplo _Desde 1 hora atrás_.
- Os resultados são armazenados em cache por 30 segundos. Use o botão de atualização para forçar uma reexecução.

## Variáveis

As variáveis permitem que um painel sirva a vários contextos, como um serviço, ambiente ou limite por vez. Eles são abordados detalhadamente no [Tutorial 3](#87-make-dashboards-reusable-with-variables).

>[!WARNING]
>
> **Limitação conhecida na compilação atual:** `{{variables}}` funciona no widget **títulos** e **widgets de texto**. No widget **SQL**, eles são renderizados no painel, mas a pré-visualização **Adicionar widget** e a verificação **Salvar** do lado do servidor rejeitam o token `{{name}}` bruto com `Unexpected character '{'`. Até que isso seja corrigido, use valores literais em consultas de widget e use **Duplicar** para criar variantes.

## Widgets de fórmula

Uma fórmula combina 2-6 consultas SQL nomeadas aritmeticamente, por exemplo `(errors / total) * 100`. Use um quando um KPI precisar de mais de uma consulta, como uma taxa, espaço, uma soma de taxas ou uma taxa de gravação. Consulte [Tutorial 3](#86-combine-queries-with-formula-widgets).

## Compartilhar, exportar e importar

| Ação | O que faz |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Salvar** | Mantém o nome, as páginas, os widgets, a disposição e as variáveis. Cada consulta de widget é revalidada no lado do servidor; um widget inválido falha em todo o salvamento. |
| **Exportar** | Baixa `<dashboard-name>.json` (guias, widgets, variáveis). |
| **Importar** | Carrega uma exportação JSON **no painel aberto**, substituindo suas guias e variáveis. Depois clique em **Salvar**. Dica: primeiro crie um painel vazio e, em seguida, importe para ele. |
| **CSV / PNG** | Downloads por widget no cabeçalho do widget. |

>[!TIP]
>
> Exportar e importar permite que você controle a versão dos painéis no Git e promova um painel de um cliente para outro. O formato de arquivo está descrito no [Apêndice](#142-export-file-format).

## Dicas para desempenho de painéis e gráficos

- **Use tabelas para conjuntos de resultados grandes** em vez de barras ou linhas de várias séries.
- **Dividir painéis ocupados em guias.** Somente os widgets da aba visível são executados.
- **Sempre `LIMIT` consultas facetadas**, especialmente em atributos de alta cardinalidade, como `urlPath`, `traceId` e `jcr.path`.
- **Filtrar com antecedência** sobre atributos de baixa cardinalidade (`service`, `kind`, `status`).
- **Prefira`TIMESERIES AUTO`** a compartimentos pequenos fixos e evite `TIMESERIES MAX`, exceto por alguns minutos.
- **Criar tendências de longo alcance para rollups.** Consulte [Desempenho](#performance).
