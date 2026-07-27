---
agent: agent
description: Analisa todos os repositórios do projeto no Azure DevOps e exporta service-map.json
---

Conecte-se ao Azure DevOps, analise todos os repositórios do projeto e mapeie as conexões entre os serviços.

## Passo 1 — Listar repositórios

Liste todos os repositórios do projeto Azure DevOps configurado.
Para cada repositório, identifique a stack tecnológica lendo os arquivos:
- `*.csproj`, `*.sln` → .NET
- `pom.xml`, `build.gradle` → Java
- `package.json` → Node.js / TypeScript
- `requirements.txt`, `pyproject.toml` → Python
- `docker-compose.yml`, `Dockerfile` → configuração de container

## Passo 2 — Extrair conexões

Para cada repositório, leia os arquivos abaixo e extraia as conexões:

**Chamadas HTTP entre serviços**
- `appsettings*.json`, `appsettings*.yml` → URLs de serviços internos
- Arquivos com `HttpClient`, `RestClient`, `WebClient`, `axios`, `requests.get`
- Variáveis de ambiente com padrão `*_URL`, `*_BASE_URL`, `*_ENDPOINT`

**Mensageria (RabbitMQ, Kafka, Azure Service Bus, SQS)**
- Configurações com `queue`, `topic`, `exchange`, `subscription`
- Código com `BasicPublish`, `Subscribe`, `SendMessage`, `Produce`, `Consume`
- Arquivos `*Consumer.cs`, `*Producer.cs`, `*Publisher.cs`, `*Handler.cs`

**Bancos de dados**
- `ConnectionStrings` em `appsettings*.json`
- Variáveis com `*_CONNECTION_STRING`, `*_DB_HOST`, `*_DB_NAME`
- Dependências nos `.csproj`: `EntityFrameworkCore`, `Dapper`, `MongoDB.Driver`, `StackExchange.Redis`, `Npgsql`

**Serviços externos**
- URLs que apontem para domínios fora da organização
- SDKs de terceiros: Stripe, SendGrid, Twilio, AWS SDK, Azure SDK

## Passo 3 — Salvar service-map.json

Após a análise, salve o arquivo `service-map.json` na raiz do workspace com a estrutura abaixo.

**Regras de mapeamento para o JSON:**
- `id` → slug do nome do repositório em kebab-case (ex: `order-service`)
- `type: "http"` + `direction: "outbound"` → chamadas HTTP para outros serviços internos
- `type: "queue"` + `direction: "publish"` → serviço publica mensagem
- `type: "queue"` + `direction: "consume"` → serviço consome mensagem
- `type: "database"` → banco de dados próprio do serviço
- Se o target for URL externa → adicionar em `externals` e referenciar com `type: "http"` apontando para o id externo

```json
{
  "services": [
    {
      "id": "order-service",
      "name": "OrderService",
      "domain": "Orders",
      "technology": ".NET 8",
      "repository": "https://dev.azure.com/org/project/_git/OrderService",
      "connections": [
        { "type": "http",     "direction": "outbound", "target": "payment-service",   "endpoint": "/api/payments",    "method": "POST" },
        { "type": "queue",    "direction": "publish",  "target": "order.created",     "broker": "RabbitMQ" },
        { "type": "queue",    "direction": "consume",  "target": "payment.confirmed", "broker": "RabbitMQ" },
        { "type": "database",                          "target": "orders-db" }
      ]
    }
  ],
  "databases": [
    { "id": "orders-db", "name": "OrdersDB", "engine": "SQL Server" }
  ],
  "queues": [
    { "id": "order.created",     "name": "order.created",     "broker": "RabbitMQ" },
    { "id": "payment.confirmed", "name": "payment.confirmed", "broker": "RabbitMQ" }
  ],
  "externals": [
    { "id": "sendgrid", "name": "SendGrid" }
  ]
}
```

## Passo 4 — Resumo

Após salvar, exiba:
- Total de serviços mapeados
- Total de conexões HTTP encontradas
- Total de filas encontradas (publish + consume)
- Total de bancos mapeados
- Lista de serviços com conexões ambíguas ou incompletas (para revisão manual)
- Caminho do arquivo salvo: `service-map.json`

> Próximos passos disponíveis: `/generate-mermaid` para diagrama de código ou `/generate-service-map` para mapa visual interativo.
