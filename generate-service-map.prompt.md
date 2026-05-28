---
mode: agent
description: Gera um arquivo service-map.html interativo a partir de um service-map.json
---

Leia o arquivo `${input:jsonPath:Caminho do service-map.json (ex: ./docs/service-map.json)}`, interprete os dados de microsserviços e gere o arquivo `service-map.html` na mesma pasta do JSON.

## Schema esperado do service-map.json

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
        { "type": "http",     "direction": "outbound", "target": "payment-service", "endpoint": "/api/payments", "method": "POST" },
        { "type": "queue",    "direction": "publish",  "target": "order.created",   "broker": "RabbitMQ" },
        { "type": "queue",    "direction": "consume",  "target": "payment.confirmed","broker": "RabbitMQ" },
        { "type": "database",                          "target": "orders-db" }
      ]
    }
  ],
  "databases": [ { "id": "orders-db",    "name": "OrdersDB",    "engine": "SQL Server" } ],
  "queues":    [ { "id": "order.created","name": "order.created","broker": "RabbitMQ"  } ],
  "externals": [ { "id": "sendgrid",     "name": "SendGrid"                            } ]
}
```

Campos obrigatórios: `services[].id`, `services[].name`, `connections[].type`, `connections[].target`.
Campos opcionais: `domain`, `technology`, `repository`, `endpoint`, `method`, `direction`, `broker`.
Valores de `type`: `http` | `queue` | `database`.
Valores de `direction`: `outbound` (http) | `publish` | `consume` (queue).

---

## Arquivo a gerar: service-map.html

HTML único, standalone, sem dependências locais. Carregar D3.js de:
`https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js`

### Visualização

- Canvas HTML5 full-screen com fundo `#0a0c10`
- Grafo força-dirigido (D3 forceSimulation)
- Formas distintas por tipo de nó:
  - **Serviço** → círculo, cor por domínio (gerar paleta automática se domínio desconhecido)
  - **Fila** → hexágono, cor `#EF9F27`
  - **Banco** → retângulo arredondado (`rrect`), cor `#639922`
  - **Externo** → círculo com borda tracejada, cor `#888780`
- Arestas por tipo:
  - **HTTP** → linha sólida + seta direcional, cor `#378ADD`
  - **Fila** → linha tracejada `[6,3]` + seta, cor `#EF9F27`
  - **Banco** → linha pontilhada `[2,3]`, cor `#639922`
- Grid de pontos no fundo em screen-space (não world-space)
- Legenda fixa no canto inferior esquerdo

### Posicionamento — comportamento crítico

- D3 force simulation com `alphaDecay: 0.022` e `velocityDecay: 0.45`
- **No evento `end` da simulação: fixar todos os nós com `node.fx = node.x` e `node.fy = node.y`**
- Arrastar um nó → manter `fx`/`fy` após soltar (posição permanente, sem voltar a balançar)
- Indicador de status: "Calculando layout..." → "Pronto — arraste para reposicionar"
- Botão **↺ Layout**: limpa `fx`/`fy` dos nós visíveis e reinicia com `sim.alpha(0.65)`

### Câmera

- Pan: arrastar canvas com mouse
- Zoom: scroll do mouse centrado no cursor
- Lerp suave por frame (fator 0.1): `cam.x += (camT.x - cam.x) * 0.1`
- Ao clicar um nó: câmera centraliza suavemente nele
- Botão Fit / tecla `F`: ajusta câmera aos nós visíveis
- DPR (`devicePixelRatio`) aplicado corretamente para telas retina

### Filtros (botão "Filtros")

- Painel flutuante com dois grupos de pills toggles:
  - **Domínios**: um pill por domínio único presente no JSON, cor = cor do domínio
  - **Tipos**: Serviços / Filas / Bancos / Externos
- Pills ativas: fundo `cor + "22"` + borda + texto colorido
- Pills inativas: fundo transparente + borda e texto dimmed
- Contador "X / Y nós visíveis" em tempo real
- Botão "Mostrar todos" reseta filtros
- Badge numérico no botão Filtros mostra filtros ativos
- Fechar ao clicar fora do painel
- Nós ocultos e suas arestas: não renderizados

### Seleção e painel lateral

- Clicar nó: seleciona, centraliza câmera, abre painel direito
- Nó selecionado: anel de glow duplo (`r+5`, `r+10`)
- Nós não conectados ao selecionado: `globalAlpha = 0.1`
- Arestas não conectadas: ocultas quando há seleção
- Arestas conectadas: partícula animada percorrendo o caminho (`pt += 0.007`)
- Painel lateral (info):
  - Nome, tech stack, domínio com cor do domínio
  - Seção "Envia para": conexões outbound com badge HTTP/pub/sub/DB
  - Seção "Recebe de": serviços que chamam este (somente serviços)
  - Seção "Usado por": para filas, bancos e externos
  - Itens de serviço clicáveis para navegar entre nós
  - Link clicável para o repositório (se presente no JSON)
  - Botão fechar (✕) + tecla Escape

### Busca

- Campo no toolbar: navega para o primeiro serviço correspondente ao texto digitado

### Toolbar

```
[🔍 Buscar...] [Filtros N] [⊡ Fit] [↺ Layout]
```

---

## Requisitos técnicos

- HTML5 + CSS3 + JavaScript vanilla (sem bundler, sem framework)
- D3.js v7 apenas para `forceSimulation` (não usar D3 para DOM)
- Renderização 100% via Canvas 2D (não SVG)
- Arquivo único, abre no browser sem servidor local
- Sem `console.log` no código gerado
- Comentários de seção no JS: `// ── NOME DA SEÇÃO ──`
- Tratar campos ausentes com fallback (domínio desconhecido → cor `#888`)
- Campos extras no JSON ignorados silenciosamente
