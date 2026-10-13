---
source-git-commit: 4949fcad799260aa73354a8c0b9d79e6394cb257
workflow-type: tm+mt
source-wordcount: '585'
ht-degree: 20%
---
## Dicionário de dados {#data-dictionary}

Isso é cada entidade e atributo que você pode consultar. O catálogo ao vivo, com descrições, também está no painel **Guia de consulta** em Explorar e na caixa de diálogo Adicionar widget.

### Rastrear entidades

| Entidade | Contém | Escopo fixo | Rollups |
| ------------- | --------------------------------------------------------- | ---------------------------- | ------------------------------------------ |
| `Transaction` | Solicitações de entrada do servidor | `kind = Server` | 1m / 5m / 1h ([Desempenho](#performance)) |
| `AppRequest` | Solicitações do consumidor de mensagens **e** do servidor de entrada | `kind IN (Server, Consumer)` | — |
| `Span` | Cada span (servidor, cliente, produtor, consumidor, interno) | — | — |

<details open>
<summary><strong>Atributos de rastreamento</strong> (idênticos em todas as três entidades)</summary>

| Atributo | Tipo | Descrição |
| --------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------- |
| `timestamp` | carimbo de data e hora | Hora de início da extensão |
| `traceId`, `spanId`, `parentSpanId` | string | Identificadores (extensões raiz têm `parentSpanId = ''`) |
| `name` | string | Nome da operação ou transação |
| `kind` | string | `Server`, `Client`, `Producer`, `Consumer`, `Internal` |
| `service` | string | Nome do serviço emissor |
| `serviceKey` | string | Identidade de serviço efetiva (retorna para `appName` quando `service` está em branco ou é um pai grosso) |
| `appName` | string | Nome do aplicativo do Observability Insights |
| `appInstance` | string | Identidade por instância: prefixo do host de criação/publicação, nome do aplicativo e serviço |
| `hostTierPrefix` | string | Nome do host truncado em seu sufixo de função `author` / `publish` / `dispatcher` |
| `duration` | número (ms) | Duração da extensão em milissegundos |
| `status` | string | `Ok`, `Error`, `Unset` |
| `statusMessage` | string | Descrição do status |
| `host` | string | Nome do host |
| `environment` | string | Ambiente de implantação |
| `region` | string | Região da nuvem |
| `cloud.provider` | string | AWS / AZURE / GCP |
| `http.method` | string | `GET`, `POST`, ... |
| `http.statusCode` | número | Status de resposta HTTP |
| `http.route` | string | Modelo de roteiro normalizado |
| `url` | string | URL completo |
| `urlPath` | string | Parte do caminho da URL (`/content/site/en.html`) |
| `externalDomain` | string | Domínio de um URL de saída (use em extensões de cliente) |
| `peer.service` | string | Serviço downstream chamado |
| `db.system`, `db.operation`, `db.namespace` | string | Tecnologia, operação e esquema do banco de dados |
| `messaging.system`, `messaging.destination` | string | Kafka, RabbitMQ, ... e tópico ou fila |
| `rpc.system`, `rpc.method` | string | gRPC e semelhante |
| `error.type` | string | Exceção ou tipo de erro |
| `sling.script`, `sling.servlet`, `sling.requestUri` | string | Detalhes do AEM Sling |
| `jsp.path` | string | JSP do AEM executado |
| `jcr.operation`, `jcr.path` | string | Operação e caminho do repositório JCR no AEM |
| `graphql.operation` | string | Nome da operação do GraphQL |

</details>

### Entidades de métrica genérica

| Entidade | Contém |
| ------------- | -------------------------------------------------- |
| `MetricGauge` | Toda amostra de medidor point-in-time OpenTelemetry |
| `MetricSum` | Toda amostra de soma cumulativa ou delta OpenTelemetry |

Use-as para **descoberta** ou para métricas que não tenham uma entidade com curadoria: `WHERE metric = 'jvm.gc.duration'`.

<details open>
<summary><strong>Atributos da métrica</strong> (compartilhados por todas as entidades de métrica)</summary>

| Atributo | Tipo | Descrição |
| ----------------------------------------------------------------------- | --------- | ------------------------------------------------------- |
| `timestamp` | carimbo de data e hora | Hora da amostra |
| `metric` | string | Nome da métrica OpenTelemetry |
| `value` | número | Exemplo de valor (convertido em entidades com curadoria; veja abaixo) |
| `unit` | string | Unidade de métrica |
| `service`, `appName`, `host`, `environment`, `region`, `cloud.provider` | string | Identidade do recurso |
| `state` | string | Modo CPU ou estado de memória |
| `cpu`, `device`, `direction`, `mountpoint` | string | Por CPU, por dispositivo, direção de I/O, sistema de arquivos |
| `pool` | string | Pool de memória JVM ou pool de conexão |
| `topic` | string | Tópico Kafka |
| `process.pid`, `process.name`, `process.owner` | string | Identidade do processo |

</details>

### Entidades de métrica preparadas

Cada entidade com curadoria envolve uma métrica OpenTelemetry com o filtro e a conversão de unidade corretos, portanto, `value` já é significativo. **Contador** entidades (✔) mantêm totais cumulativos. Consulte-os com `rate(value)` e `TIMESERIES`.

| Entidade | `value` significa | Unidade | Contador | Notas |
| ------------------------------------------------------------------------------ | ----------------------------------- | --------- | ------- | -------------------------------------------- |
| `CpuUtilization` | CPU ocupado | % |         | Acúmulo de 1 minuto para `average(value)` por host |
| `CpuUserPercent` / `CpuSystemPercent` / `CpuIowaitPercent` / `CpuStealPercent` | Tempo por modo do CPU | % |         |                                              |
| `MemoryUtilization` | Memória usada | % |         | Acúmulo de 1 minuto para `average(value)` |
| `LoadAverage1m` / `LoadAverage5m` / `LoadAverage15m` | Média de carga | — |         |                                              |
| `LoadAverageWithFallback` | 5 m de carga, recuando para 1 m | — |         |                                              |
| `DiskUtilization` | Disco usado por montagem | % |         | Faceta por `mountpoint` |
| `DiskFreeBytes` | Sistema de arquivos raiz livre | bytes |         |                                              |
| `DiskReadRate` / `DiskWriteRate` | Taxa de transferência de disco | bytes/s | ✔ |                                              |
| `DiskReadOpsRate` / `DiskWriteOpsRate` | IOPS de disco | ops/s | ✔ |                                              |
| `NetworkReceiveRate` / `NetworkTransmitRate` | taxa de transferência eth0 | bytes/s | ✔ |                                              |
| `NetworkReceiveRateAllDevices` / `NetworkTransmitRateAllDevices` | Todos os dispositivos | bytes/s | ✔ |                                              |
| `NetworkIoRate` | Todas as direções e dispositivos combinados | bytes/s | ✔ |                                              |
| `NetworkReceiveErrorRate` / `NetworkTransmitErrorRate` | Erros de rede | erros/s | ✔ |                                              |
| `NetworkReceivePacketRate` / `NetworkTransmitPacketRate` | Pacotes | pacotes/s | ✔ |                                              |
| `ProcessCpuUtilization` | CPU por processo | % |         | Faceta por `process.name` |
| `ProcessMemoryUsage` | Memória por processo | MB |         |                                              |
| `JvmHeapUsed` / `JvmHeapLimit` | heap de JVM | MB |         | Relatado por pool de memória |
| `JvmThreadCount` | Threads JVM | — |         |                                              |
| `JvmCpuRecentUtilization` | CPU do processo JVM | % |         | Acúmulo de 1 minuto |
| `DispatcherRequestRate` | Solicitações HTTP do AEM Dispatcher | solic./s | ✔ |                                              |

> [!NOTE]
> As métricas de memória JVM são relatadas **por pool de memória**, portanto, `average(value)` em `JvmHeapUsed` / `JvmHeapLimit` médias entre pools. Use `FACET host, pool` para uma exibição exata por pool.

### `SpanBreakdown`: composição de transação pré-agregada

`SpanBreakdown` contém a composição de intervalo filho por minuto de cada transação. É calculado na assimilação, portanto é barato consultar.

| Atributo | Tipo | Descrição |
| --------------- | --------- | ---------------------------------------------------------------------- |
| `timestamp` | carimbo de data e hora | Classificação de um minuto |
| `service` | string | Serviço |
| `rootOperation` | string | Nome da transação pai |
| `segment` | string | Nome da extensão secundária |
| `callCount` | número | Chamadas deste segmento no bucket |
| `avgDurationMs` | número | Duração média do segmento no bucket (defina a unidade para `ms` manualmente) |
