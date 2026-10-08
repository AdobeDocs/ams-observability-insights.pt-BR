---
title: Cobertura, ambientes e retenção de dados
description: Veja o que os Insights de observação monitoram no AEM Managed Services, como os aplicativos são representados e por quanto tempo os dados de monitoramento são retidos.
feature: Operations
role: Admin
product_v2:
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
    internal-label: Experience Manager
feature_v2:
  - id: 8a70d214-ab7b-58c1-b001-2ed2e5d6303d
    internal-label: Operations
role_v2:
  - id: c66ffd68-0f65-42bb-aa23-b4020f12e0bd
    internal-label: Admin
source-git-commit: e0cc17c9d725cad021ba99da4332bca176eae6db
workflow-type: tm+mt
source-wordcount: '267'
ht-degree: 1%
---

# Cobertura, ambientes e retenção de dados {#coverage-environments-and-data-retention}

Esta página resume quais dados são coletados nos Insights de capacidade de observação para o AEM Managed Services e como esses dados são organizados.

## Monitoramento da cobertura {#monitoring-coverage}

Monitores Adobe:

- Camadas de autor do AEM com o plug-in Java do APM dos Insights de observação
- Níveis de publicação do AEM com o plug-in Java do Insights de observação APM
- Servidores hospedados na topologia gerenciada com o agente de Infraestrutura do Observability Insights

O monitoramento personalizado de APM e infraestrutura está habilitado em ambientes Managed Services de produção e não produção.

## Como os aplicativos são representados {#how-applications-are-represented}

Cada ambiente do AEM Managed Services normalmente inclui:

- Um aplicativo APM para Autor
- Um aplicativo APM para publicação

Todas as topologias em um contrato do Managed Services se reportam em uma conta do Observability Insights.

## Retenção de dados {#data-retention}

Métricas de APM, métricas de infraestrutura e eventos relacionados são retidas por até **30 dias**.

## Tabelas de resumo {#summary-tables}

| Área de cobertura | O que é monitorado |
| -------------- | ------------------------------------------ |
| APM | Aplicativos de criação e publicação do AEM |
| Infraestrutura | Todos os servidores hospedados na topologia gerenciada |

| Item | Representação |
| ------------------------------ | ------------------------------------------------------------- |
| Ambiente do AEM | Um aplicativo APM de Autor e um aplicativo APM de Publicação |
| Conta dos Insights de observação | Uma conta gerenciada pela Adobe por escopo de cliente da Managed Services |

| Tipo de dados | Retenção |
| --------------------------------- | ------------- |
| Métricas e eventos APM | Até 30 dias |
| Métricas e eventos de infraestrutura | Até 30 dias |

## O que isso significa operacionalmente {#what-this-means-operationally}

- Os Insights de capacidade de observação são adequados para análise operacional, incidentes ativos e comparação de tendências recentes.
- A análise do histórico além da janela de retenção deve ser tratada por meio de outros relatórios ou processos de arquivamento, se necessário.
- Ao investigar problemas recorrentes, capture capturas de tela ou evidências exportadas antes que os dados expirem.
