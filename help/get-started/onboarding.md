---
title: Introdução aos Insights de observação
description: Saiba como acessar os Insights de observação, o que o Adobe monitora em seu nome e onde encontrar o que é necessário neste guia.
feature: Operations
role: Admin
source-git-commit: cc405e8b70973c33ecc6137114315998e8f9af50
workflow-type: tm+mt
source-wordcount: '533'
ht-degree: 0%

---


# Introdução aos Insights de observação {#get-started}

Esta seção aborda os aspectos essenciais para novos usuários: como acessar a conta do Observability Insights, quais ambientes e dados o Adobe monitora em seu nome e como navegar pelo restante dessa documentação.

## A interface dos Insights de observação {#observability-insights-interface}

Ao fazer logon em [insights.adobecqms.net](https://insights.adobecqms.net), a tela de abertura fornece um ponto de entrada em todas as áreas de monitoramento dos ambientes do AEM Managed Services.

![Tela de abertura do Observability Insights mostrando pontos de entrada de monitoramento de APM e Infraestrutura](../v2-assets/observability-catalog-listing.png)

A interface está organizada em torno de duas áreas principais de monitoramento:

- **Aplicativos** — Exibe os dados de desempenho do aplicativo para seus níveis de Autor e Publicação. Use-a para investigar detalhes de taxa de transferência de solicitação, taxas de erro, latência, comportamento da JVM e execução em nível de rastreamento. Consulte [Aplicativos](../applications.md).
- **Hosts** — Exibe os dados de integridade no nível do host em sua topologia gerenciada. Use isso para avaliar sinais de CPU, memória, disco, rede e armazenamento em servidores individuais. Consulte [Hosts](../hosts.md).

Ambas as áreas são somente leitura para usuários clientes. O Adobe Managed Services gerencia o provisionamento, a instrumentação e o controle administrativo da conta.

## Gerenciamento de acesso e conta {#access-overview}

O acesso aos Insights de capacidade de observação é gerenciado pelo Adobe IMS. A Adobe provisiona e gerencia a conta de sua organização; as equipes de clientes recebem acesso somente leitura a todos os dados monitorados.

Pontos principais:

- A conta de Insights de capacidade de observação de sua organização está vinculada a uma única conta principal do Adobe.
- Todos os ambientes em seu contrato do Managed Services — Autor e publicação, produção e não produção — são relatados nesta conta.
- O acesso do usuário é provisionado e gerenciado pelo seu engenheiro de sucesso do cliente (CSE).

Para obter as etapas de provisionamento, as funções dos usuários e o que os usuários clientes podem ou não fazer, consulte [Gerenciamento de acesso e conta](access-and-accounts.md).

## Cobertura, ambientes e retenção de dados {#coverage-overview}

O Adobe monitora seus níveis de Autor e Publicação do AEM usando o plug-in Java do Observability Insights APM e todos os servidores hospedados usando o agente da Infraestrutura do Observability Insights. O monitoramento é ativado em ambientes de não produção e de produção.

Pontos principais:

- Cada ambiente do AEM Managed Services inclui um aplicativo APM para Autor e um para Publicação.
- Métricas de APM, métricas de infraestrutura e eventos são retidos por até **30 dias**.
- Os Insights de capacidade de observação são adequados para análise operacional e comparação de tendências recentes; não são uma ferramenta de arquivamento ou de relatórios de longo prazo. Capture capturas de tela ou evidências exportadas antes que os dados expirem.

Para obter detalhes completos sobre a cobertura, incluindo como os aplicativos são representados em sua conta e as implicações operacionais da janela de retenção, consulte [Cobertura, ambientes e retenção de dados](coverage-and-data.md).

## Como este guia está estruturado {#how-this-guide-is-structured}

A documentação está organizada em quatro áreas. Use as descrições abaixo para ir diretamente ao que você precisa.

**Introdução** — Esta seção. Abrange acesso, provisionamento de conta, escopo de monitoramento e retenção de dados.

**[Usar Insights de Observabilidade](../use-observability-insights.md)** — Orientação orientada a tarefas para investigação diária. Use [Aplicativos](../applications.md) quando o sintoma estiver voltado para o aplicativo: páginas lentas, picos de erro ou transações instáveis. Use [Hosts](../hosts.md) quando precisar determinar se a pressão de recursos no nível do host (CPU, memória, disco ou rede) explica o que você está vendo nos Aplicativos. Os fluxos de investigação passo a passo estão disponíveis em [Investigar problemas do aplicativo](../use-cases/investigate-application-issues.md) e [Investigar problemas de infraestrutura](../use-cases/investigate-infrastructure-issues.md).

**[Perguntas Frequentes](../troubleshooting/common-questions.md)** — Perguntas comuns e pontos de entrada orientados por suporte para quando você não tiver certeza de onde começar ou precisar de respostas rápidas durante um incidente ativo.
