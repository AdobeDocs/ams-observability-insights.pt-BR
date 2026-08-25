---
title: Perguntas frequentes
description: Perguntas comuns e pontos de partida de investigação para Insights de capacidade de observação no AEM Managed Services.
feature: Operations
role: Admin
source-git-commit: 68b80f99e8be9deed37ea857d1dc7cb0ba3ec94d
workflow-type: tm+mt
source-wordcount: '186'
ht-degree: 0%

---


# Perguntas frequentes {#faq}

Use esta página como ponto de partida quando não tiver certeza de onde começar ou precisar de uma resposta rápida durante uma investigação ativa.

## Por que não posso acessar o Observability Insights? {#cannot-access-observability-insights}

Comece com [Gerenciamento de acesso e conta](../get-started/access-and-accounts.md). Se o provisionamento estiver incompleto ou desatualizado, entre em contato com o engenheiro de sucesso do cliente (CSE) para solicitar acesso ou uma atualização.

## Por que vejo &quot;Carregando permissões&quot; quando tento fazer logon? {#loading-permissions-error}

Isso normalmente indica um problema com o provisionamento do usuário. Entre em contato com o engenheiro de sucesso do cliente (CSE), que pode trabalhar com as equipes relevantes para resolver o problema de acesso.

## Como posso determinar se um problema está relacionado a aplicativos ou infraestrutura? {#application-or-infrastructure}

Comece com [Monitoramento do Desempenho de Aplicativos](/help/applications.md) para analisar taxas de solicitação, taxas de erro e latência em Autor ou Publicação. Se os sinais do aplicativo estiverem elevados, use [Hosts](/help/hosts.md) para verificar se a pressão de recursos no nível do host (CPU, memória, disco ou rede) explica ou compõe o que você está vendo.

## Quais dados os Insights de observação realmente coletam? {#what-data-is-collected}

Consulte [Cobertura, ambientes e retenção de dados](../get-started/coverage-and-data.md) para obter informações sobre escopo de monitoramento, representação de aplicativos, períodos de retenção e implicações operacionais.
