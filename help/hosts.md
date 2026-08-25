---
title: Monitoramento da infraestrutura com insights de observação
description: Saiba quando usar os painéis de infraestrutura, quais sinais analisar primeiro e onde encontrar a referência completa da métrica de host.
feature: Operations
role: Admin
source-git-commit: 825334e003ae814af1b0845c6de1a533b4b5f47b
workflow-type: tm+mt
source-wordcount: '580'
ht-degree: 0%

---


# Hosts {#hosts}

Use Hosts em Insights de Capacidade de Observação para monitorar a integridade, o desempenho e a utilização de recursos da infraestrutura que suporta seus aplicativos e serviços. Use os painéis de infraestrutura para identificar problemas relacionados à capacidade do host, pressão de armazenamento, throughput da rede ou contenção de recursos do sistema operacional.

## Que monitoramento de infraestrutura ajuda a responder {#what-infrastructure-monitoring-helps-you-answer}

O monitoramento da infraestrutura é mais útil quando você precisa responder a perguntas como:

- A lentidão do aplicativo é acompanhada por pressão de CPU, memória ou E/S?
- Um host está se comportando de forma diferente dos outros no mesmo ambiente?
- Os padrões de rede ou disco estão mudando durante o mesmo intervalo de um incidente voltado para o cliente?
- As tendências de utilização do armazenamento apontam para um problema futuro de capacidade?

## Acessando Hosts de Infraestrutura {#infrastructure-host-overview}

O Monitoramento de infraestrutura oferece visibilidade em nível de host sobre a integridade e o desempenho da infraestrutura que suporta seus ambientes AEM gerenciados. No **Catálogo de Observabilidade**, você pode navegar pelos hosts de infraestrutura e se aprofundar em um host individual para investigar CPU, memória, rede, armazenamento e outros sinais de nível de sistema.

## Acessando Hosts de Infraestrutura

Em **Catálogo**, selecione a guia **Hosts** para exibir a infraestrutura associada à conta selecionada.

![Hosts de infraestrutura](v2-assets/1_host.png)

A exibição **Hosts de infraestrutura** fornece um inventário de hosts monitorados e inclui:

- **Nome do host** — Nome do host de infraestrutura monitorada.
- **Conta** — Conta associada ao host.
- **Ambiente** — Classificação de ambiente, como `DEV` ou `STAGE`.
- **Integridade** — Status de integridade atual do host.
- **Última visualização** — Como a telemetria foi recebida recentemente do host.

![Visão geralDosHosts](v2-assets/2_hostOverview.png)

## Fluxo de investigação sugerido {#suggested-investigation-flow}

Para a maioria dos incidentes, analise o painel do host nesta ordem:

1. Verifique a utilização do CPU, a média de carga e o uso da memória para saturação óbvia.
2. Revise a espera de E/S da CPU e a taxa de transferência de disco se os tempos de resposta aumentarem sem um pico correspondente da CPU.
3. Compare a taxa de transferência da rede com o tráfego do aplicativo para identificar as alterações relacionadas à carga.
4. Verifique o uso do armazenamento e o uso do disco em nível de sistema de arquivos em busca de um risco persistente de capacidade.
5. Compare vários hosts para ver se o problema é localizado ou sistêmico.

## O que revisar primeiro {#what-to-review-first}

- **CPU e memória** quando um aplicativo parece lento ou instável em um intervalo de tempo mais amplo.
- **A E/S de disco e a E/S da CPU aguardam** quando as solicitações são interrompidas ou colocadas em fila inesperadamente.
- **E/S de rede** quando há suspeita de alteração das características de tráfego ou dependências downstream.
- **Uso do armazenamento** quando os incidentes envolvem falhas de implantação, pressão de indexação ou preocupações com capacidade a longo prazo.

Use o campo **Nome contém** e o filtro **Camada** para restringir a lista de hosts. Selecione um nome de host para abrir os detalhes de monitoramento de infraestrutura.

## Monitoramento de host

Depois de selecionar um host, a exibição **Infraestrutura** fornece páginas de monitoramento dedicadas para esse host.

A navegação do host inclui:

- **Visão geral** — Visão consolidada da integridade e dos sinais de utilização da infraestrutura principal.
- **Métricas** — Métricas detalhadas de desempenho do host, incluindo CPU, memória, carga, E/S de disco e taxa de transferência de rede.
- **Rede** — Tráfego de rede, atividade de interface e erros de transmissão/recepção.
- **Processo** — Monitoramento em nível de processo do host.
- **Armazenamento** — Utilização do disco, E/S de disco e uso do sistema de arquivos.
- **Sistema** — Métricas de recursos principais do sistema, como CPU, memória e média de carga.

## Perguntas a serem respondidas durante a investigação {#questions-to-answer}

- O problema é isolado em um host ou visível em todo o ambiente?
- O CPU, a memória ou os sinais de disco são elevados durante a mesma janela do incidente?
- O crescimento do armazenamento está tendendo para um limite que poderia afetar as operações?
- Os sintomas de infraestrutura explicam o comportamento do aplicativo ou eles só aparecem como um efeito de downstream?

## Evidências a serem capturadas durante o escalonamento {#evidence-to-capture}

- Ambiente e hosts afetados
- Janela de tempo da ocorrência
- Capturas de tela do CPU, memória, disco e rede
- Se a anomalia é isolada ou sistêmica
- Sintomas de aplicação relacionados de APM
