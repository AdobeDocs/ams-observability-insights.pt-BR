---
title: Referência do painel de infraestrutura
description: Referência painel a painel para painéis de infraestrutura de Insights de observação, incluindo capturas de tela, métricas e unidades.
feature: Operations
role: Admin
source-git-commit: 1d54a6a398360b040221db5b2780d301722894bf
workflow-type: tm+mt
source-wordcount: '1091'
ht-degree: 7%

---


# Referência do painel de infraestrutura {#infrastructure-dashboard-reference}

Esta referência documenta os painéis da infraestrutura de nível de host usados nos Insights de capacidade de observação para o AEM Managed Services.

## Visão geral do painel

O Painel de Monitoramento de Infraestrutura de Host fornece visibilidade em tempo real sobre a utilização e o desempenho do host subjacente. Essas métricas auxiliam os operadores no monitoramento de recursos de computação, memória, armazenamento e rede, ao mesmo tempo em que identificam possíveis gargalos de recursos.

O painel inclui os seguintes painéis de monitoramento:

- Utilização do CPU do host
- E/S de disco do host
- E/S de rede de host
- Espera de E/S do CPU
- Utilização do armazenamento
- Uso do disco
- Média de Carga do CPU do Host
- Uso de Memória do Host

## &#x200B;1. Utilização do CPU do host

![Utilização de CPU do Host](../assets/host-monitoring/host_cpu_utilization.png)

### Descrição

O painel **Utilização de CPU do Host** exibe a porcentagem de recursos CPU que estão sendo consumidos no momento pelo sistema operacional e por todos os processos em execução ao longo do tempo.

Essa métrica representa o uso geral do CPU no host e fornece uma visualização de série temporal da atividade do processador.

O gráfico permite que os operadores monitorem como o consumo de CPU muda durante a janela de observação selecionada.

### Métrica

| Métrica | Descrição |
| --------- | ---------------------------------------- |
| `cpu_pct` | Porcentagem do total de CPU em uso no momento |

### Unidades

- Porcentagem (%)

### Estatísticas exibidas

O painel resume a utilização do CPU usando três valores:

| Estatística | Descrição |
| --------- | --------------------------------------------------------------- |
| Média | Utilização média do CPU durante o intervalo de tempo selecionado |
| Último | Valor de utilização do CPU coletado mais recentemente |
| Max | Maior utilização de CPU observada durante o intervalo de tempo selecionado |

### Componentes do gráfico

- Linha de série temporal que representa a utilização do CPU.
- Eixo Y baseado em porcentagem variando de **0% a 100%**.
- Estatísticas de resumo exibidas abaixo do gráfico.
- Tendência histórica no intervalo de monitoramento selecionado.

## &#x200B;2. E/S de disco do host

![E/S de Disco de Host](../assets/host-monitoring/host_disk_io.png)

### Descrição

O painel **E/S de Disco do Host** exibe a taxa de transferência de armazenamento para as operações de leitura e gravação de disco realizadas pelo host.

O gráfico apresenta duas séries temporais independentes que representam os dados sendo transferidos entre o sistema operacional e os dispositivos de armazenamento.

Essa visualização ajuda a monitorar a atividade de armazenamento ao longo do tempo e fornece à insight o volume de dados que estão sendo lidos e gravados em discos.

### Métricas

| Métrica | Descrição |
| ------------ | --------------------------------- |
| `disk_read` | Quantidade de dados lidos do armazenamento |
| `disk_write` | Quantidade de dados gravados no armazenamento |

Internamente, essas métricas são exibidas usando valores de taxa de transferência suavizados.

### Unidades

- Bytes por segundo (B/s)
- Kilobytes por segundo (KB/s)
- Megabytes por segundo (MB/s)
- Gigabytes por segundo (GB/s)

A unidade exibida é dimensionada automaticamente com base na taxa de transferência.

### Componentes do gráfico

- Linha verde que representa a taxa de transferência de leitura do disco.
- Linha laranja representando taxa de transferência de gravação de disco.
- Visualização de série temporal.
- Legenda separada para cada métrica.
- Valores de métrica atuais exibidos ao lado de cada série.

## &#x200B;3. E/S de rede de host

![E/S de Rede de Host](../assets/host-monitoring/host_network_io.png)

### Descrição

O painel **E/S de Rede do Host** exibe o volume de tráfego de rede transmitido e recebido pelo host ao longo do tempo.

O gráfico mede a taxa na qual os dados fluem pelas interfaces da rede e oferece visibilidade do consumo de largura de banda da rede.
Essa métrica representa o throughput de rede agregado.

### Métrica

| Métrica | Descrição |
| --------------- | --------------------------------------------------------------------- |
| `bytes_per_sec` | Taxa de transferência de rede agregada medida como bytes transferidos por segundo |

### Unidades

O gráfico é dimensionado automaticamente entre:

- Bytes/s
- KB/s
- MB/s
- GB/s

dependendo do volume de tráfego observado.

### Estatísticas exibidas

| Estatística | Descrição |
| --------- | ---------------------------------- |
| Média | Taxa de transferência média da rede |
| Último | Medição mais recente da taxa de transferência |
| Max | Maior throughput observado |

### Componentes do gráfico

- Única linha de taxa de transferência.
- Visualização de série temporal.
- Dimensionamento dinâmico de largura de banda.
- Estatísticas resumidas exibidas abaixo do gráfico.

## &#x200B;4. Espera de E/S do CPU

![Espera de E/S do CPU](../assets/host-monitoring/cpu_io_wait.png)

### Descrição

O painel **CPU I/O Wait** exibe a porcentagem de tempo gasto no CPU aguardando a conclusão das operações de entrada/saída.

Essa métrica representa o tempo de inatividade do processador que ocorre porque os processos ativos são bloqueados enquanto aguardam dispositivos de armazenamento ou outras operações de I/O.

O gráfico visualiza como a espera de E/S muda ao longo do tempo.

### Métrica

| Métrica | Descrição |
| --------- | ------------------------------------------------------ |
| `cpu_pct` | Porcentagem de tempo gasto no CPU aguardando operações de E/S |

### Unidades

- Porcentagem (%)

### Estatísticas exibidas

| Estatística | Descrição |
| --------- | ------------------------------- |
| Média | Porcentagem média de espera de E/S do CPU |
| Último | Valor registrado mais recentemente |
| Max | Maior valor registrado |

### Componentes do gráfico

- Linha de série temporal.
- Eixo Y baseado em porcentagem.
- Estatísticas resumidas.
- Visualização de tendência histórica.

## &#x200B;5. Utilização do armazenamento

![Uso do Armazenamento](../assets/host-monitoring/storage_disk_usage.png)

### Descrição

O painel **Uso de Armazenamento** exibe a porcentagem geral da capacidade de armazenamento usada no host monitorado.

O gráfico fornece uma exibição histórica da utilização da capacidade do sistema de arquivos durante o intervalo selecionado.

### Métrica

| Métrica | Descrição |
| --------------- | -------------------------------------------------- |
| % de Uso de Armazenamento | Porcentagem de armazenamento alocado consumido no momento |

### Unidades

- Porcentagem (%)

### Componentes do gráfico

- Gráfico de utilização da série temporal.
- Escala de porcentagem.
- Tendência histórica de utilização do armazenamento.

## &#x200B;6. Uso do disco

![Uso do disco](../assets/host-monitoring/storage_disk_usage.png)

### Descrição

O painel **Uso do Disco** exibe a utilização do armazenamento para cada sistema de arquivos ou dispositivo de armazenamento montado.

Cada linha corresponde a um dispositivo de bloco específico ou partição montada e informa a porcentagem de espaço em uso no momento.

Essa tabela fornece um detalhamento da utilização do armazenamento em nível de sistema de arquivos.

### Informações exibidas

Cada entrada inclui:

| Texto | Descrição |
| --------------- | -------------------------------------------- |
| Dispositivo | Dispositivo de armazenamento ou sistema de arquivos montado |
| Usado % | Porcentagem da capacidade de armazenamento utilizada |
| Barra de Utilização | Representação visual do consumo de armazenamento |

### Unidades

- Porcentagem (%)

### Componentes do gráfico

- Lista de sistemas de arquivos/dispositivos.
- Porcentagem de utilização.
- Indicador de capacidade codificado por cores.
- Valores de utilização classificados.

## &#x200B;7. Média de Carga do CPU do Host

![Média de Carregamento do CPU do Host](../assets/host-monitoring/host_cpu_load_average.png)

### Descrição

O painel **Média de Carga do CPU do Host** exibe as médias de carga do sistema Linux em três janelas de tempo de rolagem.

Diferentemente da utilização do CPU, a média de carga representa o número médio de processos que estão sendo executados ativamente ou aguardando a programação do CPU ou a conclusão de E/S.

O gráfico exibe simultaneamente três médias contínuas que fornecem tendências de carga de trabalho de curto e longo prazo.

### Métricas

| Métrica | Descrição |
| ---------- | -------------------------------------------- |
| `load_1m` | Carga média do sistema nos últimos 1 minuto |
| `load_5m` | Carga média do sistema nos últimos 5 minutos |
| `load_15m` | Carga média do sistema nos últimos 15 minutos |

### Unidades

- Carga média (valor adimensional)

### Estatísticas exibidas

Para cada métrica de carga média:

| Estatística | Descrição |
| --------- | --------------------------------------- |
| Média | Carga média durante o intervalo de tempo selecionado |
| Último | Último valor de carga registrado |
| Max | Maior valor de carga observado |

### Componentes do gráfico

- Três linhas de tendências independentes.
- Visualização de série temporal.
- Legendas individuais para cada média variável.
- Estatísticas resumidas para cada métrica.

## &#x200B;8. Uso de Memória do Host

![Uso de Memória do Host](../assets/host-monitoring/host_memory_usage.png)

### Descrição

O painel **Uso de Memória do Host** exibe a porcentagem de memória do sistema físico atualmente alocada pelo sistema operacional.

Essa métrica representa a utilização geral da RAM em todos os processos em execução, memória do kernel, buffers e caches.

O gráfico fornece uma exibição contínua do consumo de memória durante todo o período de monitoramento selecionado.

### Métrica

| Métrica | Descrição |
| ------------ | ---------------------------------------------- |
| `memory_pct` | Porcentagem de memória física em uso no momento |

### Unidades

- Porcentagem (%)

### Estatísticas exibidas

| Estatística | Descrição |
| --------- | ---------------------------------- |
| Média | Utilização média da memória |
| Último | Utilização registrada mais recentemente |
| Max | Maior utilização observada |

### Componentes do gráfico

- Gráfico de utilização da memória de série temporal.
- Eixo Y baseado em porcentagem.
- Tendência de utilização histórica.
- Estatísticas resumidas.

## Resumo das métricas do painel

| Painel do painel | Métrica primária | Unidade |
| --------------------- | -------------------------------- | ------------ |
| Utilização do CPU do host | `cpu_pct` | % |
| E/S de disco do host | `disk_read`, `disk_write` | Bytes/s |
| E/S de rede de host | `bytes_per_sec` | Bytes/s |
| Espera de E/S do CPU | `cpu_pct` | % |
| Utilização do armazenamento | % de Uso de Armazenamento | % |
| Uso do disco | Uso do sistema de arquivos | % |
| Média de Carga do CPU do Host | `load_1m`, `load_5m`, `load_15m` | Média de carga |
| Uso de Memória do Host | `memory_pct` | % |
