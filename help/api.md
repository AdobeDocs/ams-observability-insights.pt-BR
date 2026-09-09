---
source-git-commit: e5523081fcd68500602e5d1bf853694d1f6c3980
workflow-type: tm+mt
source-wordcount: '1104'
ht-degree: 7%

---
# API pública dos Insights de capacidade de observação

A API pública do Observability Insights permite que você insira seus próprios dados de observabilidade — visões gerais de solicitações, catálogos de serviço, rastreamentos e métricas — diretamente em suas próprias ferramentas, scripts e painéis.

- **URL Base da API (API_BASE_URL):** `https://insights.adobecqms.net/`
- **Formato:** JSON sobre HTTPS
- **Autenticação:** chave de API (token de portador)

> Substitua `{{API_BASE_URL}}` em todo este documento pelo host da API da instância do Observability Insights, por exemplo, `https://insights.adobecqms.net/`.

&#x200B;---

## &#x200B;1. Obter uma chave de API

As chaves de API são credenciais pessoais vinculadas à sua conta e limitadas a uma única organização. Uma chave só pode ler dados de locatários que pertencem à organização para a qual foi criada; ela nunca pode ver os dados de outra organização.

### Gerar uma chave

1. Faça logon no [painel do Observability Insights](https://insights.adobecqms.net/).
2. Abra o menu de perfil (canto superior direito) → **Chaves de API**.
   ![Menu Chaves de API](v2-assets/api-key.png)
3. Na guia **Chaves de API**, clique em **Gerar chave**.
   ![Gerar chave de API](v2-assets/api-key-gen.png)
4. Dê a ele um nome descritivo (por exemplo, `CI pipeline`, `Grafana datasource`), escolha a organização para a qual ele deve ter escopo e, opcionalmente, defina uma data de expiração.
5. Clique em **Gerar chave**. Sua chave é mostrada **uma vez**, no formato:

   ```
   synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
   ```

   **Copie-o imediatamente e armazene-o em um local seguro** (um gerenciador de segredos, armazenamento de segredo CI, etc.) — o painel não poderá mostrá-lo novamente. Se você perdê-la, revogue-a e gere uma nova.

### Gerenciar chaves existentes

A seção Chaves de API lista todas as chaves criadas, incluindo a organização, a data de criação, a expiração e o carimbo de data e hora da última utilização. Clique no ícone de lixeira ao lado de uma chave para **revogá-la** — a revogação é imediata e não pode ser desfeita.

### Segurança de chave

- Trate uma chave de API exatamente como uma senha. Qualquer pessoa com a chave pode ler todos os dados de observabilidade de cada locatário na organização para a qual tem escopo, até que seja revogada ou expire.
- Nunca confirme uma chave para o controle de origem ou compartilhe-a em texto sem formatação (bate-papo, email, tíquetes).
- Gire as chaves periodicamente e revogue qualquer chave que não esteja mais em uso.
- Se uma chave estiver comprometida, revogue-a imediatamente de **Configurações da Organização → Chaves da API** e gere uma substituição.

&#x200B;---

## &#x200B;2. Solicitações de autenticação

Cada solicitação para a API pública deve incluir sua chave no cabeçalho `Authorization`:

```
Authorization: Bearer synx_9pQ2v6f1WYbLZk3n0aRtEo4jXcHsVmDgUiPq7B8l1yc
```

Solicitações sem uma chave válida ou com uma chave expirada/revogada recebem `401 Unauthorized`. Os logons da sessão (cookies/tokens do navegador) **não** foram aceitos nesta API.

&#x200B;---

## &#x200B;3. Conceitos básicos

### Inquilinos

Cada ponto de extremidade requer um parâmetro de consulta `tenant_id` identificando os dados do locatário a serem lidos. Uma chave só pode consultar locatários que pertençam à organização para a qual foi criada; solicitar um locatário fora dessa organização retorna `403 Forbidden`. Não há nenhum modo &quot;todos os locatários&quot; nesta API — sempre passe um `tenant_id` específico.

Não tem certeza de quais `tenant_id` valores sua chave pode usar? Chamada [`GET /public/v1/tenants`](#get-publicv1tenants) — lista exatamente os locatários que sua chave está autorizada a consultar.

### Intervalos de tempo

Os pontos de extremidade que aceitam parâmetros `from` / `to` usam carimbos de data/hora Unix (segundos), carimbos de data/hora de milissegundos ou cadeias de caracteres de data/hora ISO 8601, por exemplo:

```
from=1735689600
from=2025-01-01T00:00:00Z
```

Se omitido, a maioria dos endpoints assumirá como padrão uma janela contínua recente (consulte cada endpoint abaixo).

### Limites de taxa

As solicitações são limitadas pela taxa por chave de API. Se você exceder o limite, receberá:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60

{ "error": "Too Many Requests", "message": "Rate limit of 300 requests/60s exceeded" }
```

Retorne e tente novamente após o número de segundos no cabeçalho `Retry-After`. Entre em contato com o suporte se o caso de uso precisar de um limite mais alto.

### Erros

Os erros são retornados como JSON com um campo `error` e, geralmente, um `message` legível por humanos:

```json
{ "error": "Bad Request", "message": "tenant_id is required" }
```

| Status | Significado |
| ------------------------- | ------------------------------------------------------------------ |
| `400 Bad Request` | Parâmetro ausente ou inválido (ex: não `tenant_id`, intervalo de tempo incorreto) |
| `401 Unauthorized` | Chave de API ausente, inválida, expirada ou revogada |
| `403 Forbidden` | A chave não está autorizada para o locatário solicitado |
| `429 Too Many Requests` | Limite de taxa excedido — consulte `Retry-After` |
| `502 Bad Gateway` | Falha na consulta upstream — é seguro tentar novamente |
| `503 Service Unavailable` | Infraestrutura de dados temporariamente indisponível |

&#x200B;---

## &#x200B;4. Pontos de acesso

### `GET /public/v1/tenants`

Lista as IDs de locatário que sua chave está autorizada a consultar. Chame isso primeiro — todos os outros pontos de extremidade exigem um desses valores como `tenant_id`.

```bash
curl -s "{{API_BASE_URL}}/public/v1/tenants" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{ "tenants": ["tenant1", "tenant2"] }
```

### `GET /public/v1/overview`

KPIs de integridade de alto nível para um locatário em uma janela de tempo: volume de solicitação, taxa de erro e percentis de latência.

| Param | Obrigatório | Descrição |
| ------------ | -------- | ------------------------------------------------------------------------- |
| `tenant_id` | Sim | Inquilino a consultar |
| `from`, `to` | Não | Intervalo de tempo (consulte [Intervalos de tempo](#time-ranges)) |
| `minutes` | Não | Encurtar para &quot;últimos N minutos&quot; se `from`/`to` não forem fornecidos (padrão `15`) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/overview?tenant_id=<tenant_id>&minutes=30" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "total_spans": 48213,
  "errors": 112,
  "error_rate_pct": 0.23,
  "p50_ms": 34,
  "p95_ms": 210,
  "p99_ms": 480,
  "service_count": 12,
  "trace_count": 9021
}
```

### `GET /public/v1/services`

Lista nomes de serviço distintos relatados para um locatário.

| Param | Obrigatório | Descrição |
| ------------ | -------- | --------------------------------------------------------------------- |
| `tenant_id` | Sim | Inquilino a consultar |
| `from`, `to` | Não | Restringir aos serviços vistos nessa janela; o padrão é os últimos 7 dias |

```bash
curl -s "{{API_BASE_URL}}/public/v1/services?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "services": ["checkout-api", "payments-worker", "web-frontend"]
}
```

### `GET /public/v1/traces`

Pesquisa rastreamentos recentes de um locatário, com filtros opcionais.

| Param | Obrigatório | Descrição |
| ----------------- | -------- | ------------------------------------------------- |
| `tenant_id` | Sim | Inquilino a consultar |
| `from`, `to` | Não | Intervalo de tempo; o padrão é durar 24 horas |
| `limit` | Não | Máximo de linhas a serem retornadas (1-200, padrão 100) |
| `offset` | Não | Deslocamento de paginação (padrão 0) |
| `service` | Não | Filtrar por nome de serviço |
| `app_name` | Não | Filtrar por nome de aplicativo/instância |
| `status` | Não | Filtrar por status de rastreamento: `ok`, `error` ou `unset` |
| `search` | Não | Pesquisa de texto livre em nomes de extensão/operação |
| `min_duration_ms` | Não | Somente rastreamentos nesta duração ou acima dela |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces?tenant_id=<tenant_id>&status=error&limit=25" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TraceId": "4bf92f3577b34da6a3ce929d0e0e4736",
      "ServiceName": "checkout-api",
      "DurationMs": 812,
      "StatusCode": "Error",
      "Timestamp": "2026-08-30T09:12:44Z"
    }
  ],
  "rows": 137,
  "limit": 25,
  "offset": 0
}
```

Use `rows` (a contagem total correspondente) com `limit`/`offset` para percorrer os resultados.

### `GET /public/v1/traces/:traceId`

Retorna a cascata de extensão completa para um único rastreamento.

| Param | Obrigatório | Descrição |
| ----------- | -------- | ---------------------------------------- |
| `tenant_id` | Sim | Locatário ao qual o rastreamento pertence |
| `limit` | Não | Máximo de extensões a serem retornadas (1-500, padrão 500) |
| `offset` | Não | Deslocamento de paginação para rastreamentos muito grandes |

```bash
curl -s "{{API_BASE_URL}}/public/v1/traces/4bf92f3577b34da6a3ce929d0e0e4736?tenant_id=<tenant_id>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "spans": [
    {
      "SpanId": "00f067aa0ba902b7",
      "Name": "POST /checkout",
      "DurationMs": 812,
      "children": []
    }
  ],
  "totalDurationMs": 812,
  "spanCount": 14,
  "limit": 500,
  "offset": 0
}
```

### `GET /public/v1/metrics`

Retorna pontos de dados de métrica bruta para um locatário.

| Param | Obrigatório | Descrição |
| ---------------------------------- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `tenant_id` | Sim | Inquilino a consultar |
| `metric` | Um de `metric`/`like` | Nome exato da métrica |
| `like` | Um de `metric`/`like` | Padrão SQL `LIKE` que deve corresponder a vários nomes de métrica |
| `type` | Não | `gauge` (padrão) ou `sum` |
| `from`, `to` | Não | Intervalo de tempo; o padrão é durar 24 horas |
| `service` | Não | Filtrar por nome de serviço |
| `host` | Não | Filtrar por nome de host. Necessário para as métricas de host de infraestrutura abaixo — sem ele, as leituras de cada host no locatário são combinadas |
| `attribute_key`, `attribute_value` | Não | Filtrar por um atributo de métrica específico (deve ser usado junto) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=jvm.memory.used&type=gauge" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "data": [
    {
      "TimeUnix": "2026-08-30T09:00:00Z",
      "MetricName": "jvm.memory.used",
      "Value": 512482816,
      "ServiceName": "checkout-api",
      "host": ""
    }
  ],
  "rows": 1
}
```

#### Métricas de host de infraestrutura

O mesmo endpoint também atende às métricas em nível de host mostradas no painel de Infraestrutura (CPU, memória, média de carga, E/S de disco, E/S de rede). Use estas combinações exatas de `metric` / `attribute_key` / `attribute_value`, sempre com um `host`:

| Widget de Painel | `metric` | `attribute_key` | `attribute_value` |
| --------------------- | ------------------------------------- | --------------- | ------------------------------------------------------------------------------------------- |
| CPU % | `system.cpu.utilization` | `state` | `idle` (subtrair de 1 para &quot;em uso&quot;), ou consulta `user`/`system`/`iowait` separadamente e soma |
| Uso de memória % | `system.memory.utilization` | `state` | `used` |
| Carga média (1m) | `system.cpu.load_average.1m` | — | — |
| E/S de leitura de disco | `system.disk.io` (`type=sum`) | `direction` | `read` |
| E/S de gravação de disco | `system.disk.io` (`type=sum`) | `direction` | `write` |
| Operações de leitura de disco | `system.disk.operations` (`type=sum`) | `direction` | `read` |
| Operações de gravação de disco | `system.disk.operations` (`type=sum`) | `direction` | `write` |
| Entrada de rede | `system.network.io` (`type=sum`) | `direction` | `receive` |
| Saída de rede | `system.network.io` (`type=sum`) | `direction` | `transmit` |

```bash
curl -s "{{API_BASE_URL}}/public/v1/metrics?tenant_id=<tenant_id>&metric=system.cpu.utilization&type=gauge&attribute_key=state&attribute_value=idle&host=<host_name>" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

**Importante — os valores de disco e de rede são contadores brutos e em constante aumento, não taxas.** Os gráficos de &quot;bytes/s&quot; e &quot;operações/s&quot; do painel são computados fazendo-se duas leituras consecutivas do contador e dividindo-se pelo tempo decorrido:

```
rate = (value_at_t2 - value_at_t1) / (t2 - t1_in_seconds)
```

### `GET /public/v1/pages`

Principais páginas de conteúdo solicitadas (`.html`) por instância do Dispatcher, classificadas por contagem de solicitações. Com o suporte da métrica `dispatcher.httpd.requests` — esse endpoint é específico para logs de acesso no estilo do AEM Dispatcher/CDN, não uma ferramenta geral de análise de página.

| Param | Obrigatório | Descrição |
| ------------ | -------- | -------------------------------------- |
| `tenant_id` | Sim | Inquilino a consultar |
| `from`, `to` | Não | Intervalo de tempo; o padrão é durar 24 horas |
| `limit` | Não | Máximo de linhas a serem retornadas (1-500, padrão 50) |

```bash
curl -s "{{API_BASE_URL}}/public/v1/pages?tenant_id=<tenant_id>&limit=50" \
  -H "Authorization: Bearer $Observability_Insights_API_KEY"
```

```json
{
  "tenant_id": "<tenant_id>",
  "from": 1735689000,
  "to": 1735690800,
  "data": [
    {
      "instance": "<instance_name>",
      "domain": "www.abc.com",
      "path": "/join-us/insights.html",
      "full_url": "https://www.abc.com/join-us/insights.html",
      "requests": 7
    }
  ],
  "rows": 1
}
```

&#x200B;---

## &#x200B;5. O que essa API não faz

- **Nenhum acesso SQL bruto.** Todos os endpoints retornam formas de dados selecionadas e criadas com propósitos específicos; não é possível consultar diretamente o armazenamento de dados subjacente.
- **Nenhuma consulta entre locatários.** Cada solicitação tem como escopo exatamente um `tenant_id`.
- **Sem acesso de gravação.** A API pública é somente leitura.

&#x200B;---

## &#x200B;6. Suporte

Se você encontrar erros inesperados ou tiver um caso de uso não coberto por esses endpoints, entre em contato com o engenheiro de sucesso/ativação do cliente para obter ajuda adicional.
