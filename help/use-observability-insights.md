---
title: Usar insights de observação
description: Entenda as quatro principais experiências de monitoramento e investigação em Insights de capacidade de observação e quando usar cada uma.
feature: Operations
role: Admin
source-git-commit: 6bbc906fa1c5570bc7ee2a6f536dd806c0c0db41
workflow-type: tm+mt
source-wordcount: '426'
ht-degree: 0%

---


# Usar insights de observação {#use-observability-insights}

Esta seção aborda os workflows diários de monitoramento e investigação que sua equipe mais usa, organizados em duas áreas de monitoramento: monitoramento do desempenho de aplicativos e monitoramento da infraestrutura.

## A interface dos Insights de observação {#observability-insights-interface}

O painel de navegação esquerdo dos Insights de capacidade de observação fornece acesso a todas as áreas de monitoramento de seus ambientes do AEM Managed Services.

![Interface do Observability Insights mostrando a navegação à esquerda com opções de APM e Infraestrutura e o painel de Monitoramento de Infraestrutura com métricas de host e filtros de ambiente](v2-assets/navigation-panel-desc.png)

A navegação inclui:

- **Catálogo** — Inventário central de aplicativos e hosts monitorados do AEM. Procure recursos nos níveis de **Autor, Publicação e Dispatcher**, com indicadores principais de integridade e desempenho, como tempo de resposta, taxa de transferência, taxa de erro e Apdex.

- **Explorar** — Investigue a telemetria de observabilidade e analise os dados de desempenho nos recursos monitorados.

- **Rastreamentos** — Analise transações de aplicativo de ponta a ponta e solicite rastreamentos para identificar latência, erros e gargalos de desempenho.

- **Painéis** — Acesse painéis com curadoria para uma visualização mais profunda e para o monitoramento de sinais de aplicativos e infraestrutura.

Os recursos podem ser filtrados por conta e nível, enquanto o catálogo fornece uma visualização consolidada da integridade do aplicativo e do host na topologia gerenciada do AEM.

## Aplicativos{#applications}

Use os [Aplicativos](applications.md) quando o problema estiver voltado para o aplicativo — páginas lentas, taxas de erro crescentes, transações instáveis ou latência inesperada no Author ou Publish.

Os aplicativos ajudam a responder:

- O problema é isolado para Autor, Publicação ou afeta ambos os níveis?
- Quais endpoints ou transações contribuem mais para o tráfego e a desaceleração?
- A latência ou os erros foram alterados antes ou depois de uma implantação ou pico de tráfego?
- Os rastreamentos apontam para operações de repositório, dependências externas ou pressão de JVM?

Os aplicativos instrumentam as transações do AEM para chamadas de método, dependências externas e operações de repositório, de modo que você possa passar rapidamente de um sintoma amplo para um caminho de execução específico.

## Hosts {#hosts}

Use [Hosts](hosts.md) quando precisar determinar se o comportamento do aplicativo é causado ou agravado pelas condições de recurso do host — saturação de CPU, pressão de memória, E/S de disco, taxa de transferência de rede ou capacidade de armazenamento.

O monitoramento de host ajuda a responder:

- A lentidão do aplicativo é acompanhada por pressão de CPU, memória ou E/S no nível do host?
- Um host está se comportando de forma diferente dos outros no mesmo ambiente?
- As tendências de utilização de disco ou armazenamento apontam para um problema futuro de capacidade?
- Os padrões de infraestrutura explicam o comportamento do aplicativo ou são um efeito de downstream?

Use painéis de host ao lado de aplicativos para distinguir entre regressões no nível do aplicativo e restrições de recursos no nível do ambiente.

Ambos os artigos incluem fluxos de trabalho de investigação, perguntas para orientar sua triagem e uma lista de evidências para capturar ao encaminhar para o Adobe Managed Services.
