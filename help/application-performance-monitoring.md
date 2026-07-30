---
title: Monitoramento do desempenho de aplicativos (APM) com o Synoptryx
description: Use o plug-in APM do Synoptryx para rastrear transações do AEM, monitorar a JVM, analisar transações e inspecionar rastreamentos de transações e serviços externos no AEM Managed Services.
feature: Operations
role: Admin
source-git-commit: 883b68e3bc57ba6b55559560a967a6dbc553262a
workflow-type: tm+mt
source-wordcount: '1089'
ht-degree: 5%

---


# Monitoramento do desempenho de aplicativos (APM) com o Synoptryx {#application-performance-monitoring}

O Monitoramento de desempenho de aplicativo (APM) do Synoptryx fornece insight histórico e em tempo real para o desempenho do Adobe Experience Manager (AEM) e a experiência do usuário final. Rastreamento de transações de ponta a ponta, gráficos e relatórios dão visibilidade do comportamento do aplicativo até o nível do código Java.

## Plug-in APM do Managed Services Synoptryx {#apm-plugin}

O AEM é executado como um aplicativo Java no Jetty com módulos OSGi Apache Felix, criados no Apache Sling e no Jackrabbit Oak. Adobe Managed Services, AEM Engineering e Synoptryx Engineering desenvolveram em conjunto instrumentação personalizada para ambientes Managed Services.

Essa instrumentação recolhe:

- **Nomeação de transação significativa** — as extensões Sling alinham nomes de transação com a estrutura da página e adicionam um atributo `requestURL` aos eventos de Insights para que você possa correlacionar URLs do Sling entre painéis.

![Exibição do rastreamento APM do Synoptryx mostrando um nome de transação descritivo do AEM com rota de verificação de integridade do Sling e linha do tempo de span](assets/image19a.png)

- **Instrumentação JCR** — As operações no nível do repositório (incluindo XPath e JCR-SQL2) são categorizadas e anexadas a rastreamentos de transações na seção de banco de dados do APM.

![Exibição do rastreamento APM do Synoptryx mostrando extensões de componentes aninhados do AEM e linha do tempo de execução para uma solicitação de página](assets/image19.png)

## Uso do Synoptryx APM {#using-apm}

Use o APM para encontrar problemas de aplicativos antes que eles afetem os usuários finais. Crie e publique compartilham uma base de código, mas são monitorados como **aplicativos APM separados** para que você possa analisar cada camada de maneira independente.

Todo ambiente do Managed Services inclui:

- Um aplicativo APM para Autor
- Um aplicativo APM para publicação

Selecione um nome de aplicativo no Synoptryx APM para abrir sua visão geral e o painel de monitoramento.

![Lista de aplicativos APM do Synoptryx mostrando os aplicativos do Author e Publish](assets/image1a.png)

## Seções do painel

O painel Gerenciamento de Desempenho de Aplicativos contém as seguintes seções:

- Visão geral
- Métricas RED (Taxa · Erros · Duração)
- Tráfego
- Latência e desempenho
- Detalhes do erro
- Principais Transações
- Integridade de JVM
- Memória JVM
- Coleta de lixo

Somente as seções mostradas abaixo estão documentadas neste guia.

## Navegação do painel

![Navegação no painel](assets/apm/1_opening_screen.png)

O painel é organizado em seções expansíveis que agrupam métricas de desempenho do aplicativo relacionadas. Expandir uma seção revela um ou mais gráficos associados a essa categoria.

## Visão geral

![Visão geral](assets/apm/1.1_apm_overview.png)

### Descrição

A seção **Visão Geral** apresenta KPIs (Indicadores-Chave de Desempenho) de alto nível resumindo o estado atual do aplicativo monitorado.

Esses KPIs fornecem um resumo instantâneo da atividade do aplicativo, da taxa de transferência, do sucesso da solicitação e da experiência geral do usuário.

### Métricas

#### Total de solicitações

Exibe o número total de solicitações processadas pelo aplicativo durante o intervalo de tempo selecionado.

**Métrica**

```
total_requests
```

**Unidade**

- Contagem

#### Taxa de transferência atual

Exibe a taxa de processamento da solicitação atual.

**Métrica**

```
throughput
```

**Unidade**

- Solicitações por segundo (solic./s)

#### Taxa de erro atual

Exibe o percentual de solicitações que resultam em erros.

**Métrica**

```
error_rate
```

**Unidade**

- Porcentagem (%)

#### Pontuação APDEX

Exibe o Índice de Desempenho de Aplicativos (APDEX), uma medida padronizada da satisfação do usuário final com base nos tempos de resposta do aplicativo.

O limite configurado é exibido dentro do widget.

**Métrica**

```
apdex_score
```

**Unidade**

- Pontuação (0,0 - 1,0)

## Métricas de VERMELHO

A metodologia RED mede três características principais de um pedido:

- **Taxa**
- **Erros**
- **Duração**

### Taxa de solicitações

![Taxa de Solicitação](assets/apm/2_red_metrics_request_rate.png)

#### Descrição

Exibe o número de solicitações de aplicação recebidas ao longo do tempo.

Este gráfico representa a taxa de transferência de solicitações usando uma visualização de série temporal.

#### Métrica

```
req_min
```

#### Unidade

- Solicitações por minuto (solic./m)

#### Informações exibidas

- Taxa de solicitações de série temporal
- Atividade de solicitação histórica
- Tendência da taxa de solicitações
- Legenda da métrica

### Taxa de erro

![Taxa de erro](assets/apm/3_error_rate.png)

#### Descrição

Exibe a porcentagem de solicitações que resultaram em erros.

O gráfico compara porcentagens de erro históricas e atuais.

#### Métricas

```
error_pct (now)
error_pct (1h ago)
```

#### Unidade

- Porcentagem (%)

#### Informações exibidas

- Porcentagem de erros atual
- Comparação histórica
- Valores médios
- Tendência de série temporal

### Duração da solicitação

![Duração da Solicitação](assets/apm/4_request_duration_p50_p95.png)

#### Descrição

Exibe a latência da solicitação em vários percentis de tempo de resposta.

O gráfico representa simultaneamente as medidas de latência do percentil coletadas durante o período de observação selecionado.

#### Métricas

```
P50
P75
P90
```

#### Unidades

- Milissegundos (ms)
- Segundos (s)

As unidades são dimensionadas automaticamente dependendo da duração da resposta.

#### Estatísticas exibidas

Para cada percentil:

- Média
- Último
- Máximo

#### Definições de percentil

| Métrica | Descrição |
| ------ | ----------------------------- |
| P50 | 50º percentil de tempo de resposta |
| P75 | 75º percentil de tempo de resposta |
| P90 | 90º percentil de tempo de resposta |

## Tráfego

### Solicitações por código de status HTTP

![Solicitações por código de status](assets/apm/5_requests_by_status_code.png)

#### Descrição

Exibe o throughput da solicitação agrupado por código de status de resposta HTTP.

Cada código de status é representado de forma independente ao longo do tempo.

#### Métricas

As métricas comuns incluem:

```
req_s 200
req_s 300
req_s 400
req_s 500
```

dependendo da atividade do aplicativo.

#### Unidade

- Solicitações por segundo (solic./s)

#### Informações exibidas

- Taxa de transferência por status HTTP
- Taxa de transferência média
- Taxa de transferência mais recente
- Taxa de transferência máxima
- Atividade de série temporal

### Taxa de solicitações por ponto de extremidade

![Taxa de Solicitação por Ponto de Extremidade](assets/apm/6_request_rate_by_end_point.png)

#### Descrição

Exibe os pontos finais de aplicativo de tráfego mais alto classificados por taxa de solicitação.

Cada ponto de extremidade é exibido como uma barra horizontal representando o volume da solicitação.

#### Métrica

```
endpoint_request_rate
```

#### Unidade

- Solicitações por minuto (solic./m)

#### Informações exibidas

- Caminho do ponto de extremidade
- Taxa de solicitações
- Lista de pontos de extremidade classificados
- Volume de solicitações relativo

## Latência e desempenho

### Tempo de resposta — P95 versus 1 hora

![Tempo de resposta P95](assets/apm/7_response_time_p95_1h.png)

#### Descrição

Exibe uma comparação do tempo de resposta atual do P95 com o tempo de resposta do P95 registrado uma hora antes.

Ambos os conjuntos de dados são exibidos no mesmo gráfico de séries temporais.

#### Métricas

```
P95 (Current)
P95 (1 Hour Ago)
```

#### Unidades

- Milissegundos (ms)
- Segundos (s)

#### Estatísticas exibidas

- Média
- Último
- Máximo

### Pontuação APDEX ao longo do tempo

![APDEX](assets/apm/8_apdex_score_overtime.png)

#### Descrição

Exibe o Índice de Desempenho da Aplicação como uma série de tempo contínua.

O gráfico visualiza os valores de APDEX em todo o intervalo de monitoramento selecionado.

#### Métrica

```
APDEX Score
```

#### Unidade

- Pontuação (0,0-1,0)

#### Estatísticas exibidas

- Média
- Último
- Máximo

### Taxa de transferência vs latência P95

![Taxa de transferência vs. latência](assets/apm/9_throughput_vs_p95latency.png)

#### Descrição

Exibe a taxa de transferência da solicitação e a latência de resposta P95 na mesma linha do tempo.

O gráfico permite a visualização simultânea do volume de tráfego e da latência de resposta.

#### Métricas

```
Throughput
P95 Latency
```

#### Unidades

| Métrica | Unidade |
| ----------- | ------------ |
| Taxa de transferência | Solicitações/s |
| Latência P95 | Milissegundos |

#### Informações exibidas

- Taxa de transferência de série temporal
- Latência de série temporal
- Comparação de métricas duplas

## Detalhes do erro

### % de Taxa de Erro por Grupo de Status

![Taxa de Erros por Grupo de Status](assets/apm/10_error_rate_pct_by_status_group.png)

#### Descrição

Exibe porcentagens de erro de aplicativo agrupadas por classe de resposta HTTP.

São representadas séries separadas para cada categoria de resposta.

#### Métricas

Os grupos comuns incluem:

```
2xx
3xx
4xx
5xx
Combined Error Trend
```

dependendo do tráfego observado.

#### Unidade

- Porcentagem (%)

#### Informações exibidas

- Porcentagem de erros por classe de resposta
- Porcentagem média de erros
- Tendência de série temporal


### Tendência da taxa de erro — Agora versus 1 hora atrás

![Taxa de Erro 1 Hora](assets/apm/11_error_ratio_trend_1h.png)

#### Descrição

Exibe a taxa de erros do aplicativo atual junto com a taxa de erros registrada uma hora antes.

#### Métricas

```
Current Error Ratio
1 Hour Error Ratio
```

#### Unidade

- Porcentagem (%)

#### Informações exibidas

- Tendência atual
- Comparação histórica
- Visualização de série temporal

### Tendência da taxa de erro — Agora versus 6 horas atrás

![Taxa de Erro 6 Horas](assets/apm/12_error_ratio_trend_6h.png)

#### Descrição

Exibe a taxa de erros do aplicativo atual junto com a taxa de erros registrada seis horas antes.

#### Métricas

```
Current Error Ratio
6 Hour Error Ratio
```

#### Unidade

- Porcentagem (%)

#### Informações exibidas

- Taxa de erro atual
- Comparação histórica
- Visualização de série temporal

## Resumo das métricas do painel

| Painel | Métricas principais |
| -------------------------- | --------------------------------------------- |
| Visão geral | Total de solicitações, throughput, taxa de erro, APDEX |
| Taxa de solicitações | Solicitações por minuto |
| Taxa de erro | Porcentagem de erros |
| Duração da solicitação | Latência P50, P75, P90 |
| Solicitações por código de status | Throughput do Status HTTP |
| Taxa de solicitações por ponto de extremidade | Volume de Solicitações de Ponto de Extremidade |
| Comparação de tempo de resposta | P95 atual vs. histórico |
| Pontuação APDEX | Índice de satisfação do usuário |
| Taxa de transferência vs. latência | Throughput de Solicitações e Latência P95 |
| Taxa de erro por grupo de status | Porcentagem de Erros do Grupo de Status HTTP |
| Tendências da Taxa de Erro | Taxa de erro atual vs. histórico |
