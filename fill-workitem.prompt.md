---
agent: agent
description: Preenche título e descrição de um work item do Azure Boards a partir de um contexto, usando o MCP do Azure DevOps
---

Você tem acesso ao servidor MCP do Azure DevOps. Use as ferramentas desse servidor (ex.: `wit_get_work_item`, `wit_create_work_item`, `wit_update_work_item`, `core_list_projects`) para consultar, criar e atualizar work items no Azure Boards. Não peça ao usuário para preencher os campos manualmente na interface — a alteração deve ser feita via MCP.

**Contexto:** ${input:contexto:Descreva o contexto do work item (requisito, bug, trecho de conversa, PR, código, etc.)}

**Work item a atualizar:** ${input:workItemId:ID do work item existente (deixe em branco para criar um novo)}

## Passo 1 — Entender o contexto e identificar o alvo

Analise o contexto informado e determine:

- **Tipo do work item**: Bug, User Story / Product Backlog Item, Task ou Feature. Se não for possível inferir com segurança, pergunte ao usuário antes de prosseguir.
- **Ação**: atualizar um work item existente ou criar um novo.
  - Se `workItemId` foi informado (ou o contexto menciona um ID/link do Azure Boards), use `wit_get_work_item` para carregar o item atual — tipo, título, descrição e demais campos preenchidos — antes de editar.
  - Se não houver ID, é necessário criar um novo item. Confirme **projeto** e, se aplicável, **area path** / **iteration path** com o usuário antes de criar; use `core_list_projects` se o projeto não estiver claro pelo contexto.

## Passo 2 — Gerar o título (`System.Title`)

- Curto e específico (até ~100 caracteres), sem ponto final.
- Para User Story/PBI/Feature: descreva o valor entregue (ex.: "Permitir exportação de relatório em PDF").
- Para Bug: descreva o sintoma observável, não a causa (ex.: "Exportação de relatório falha com arquivos acima de 50MB").
- Para Task: comece com um verbo de ação (ex.: "Configurar pipeline de build para o serviço X").
- Evite termos genéricos como "Ajuste", "Melhoria" ou "Correção" sem qualificar o que muda.

## Passo 3 — Gerar a descrição (`System.Description`)

Monte a descrição em HTML (formato aceito pelo campo rich-text do Azure Boards), com as seções pertinentes ao tipo do work item:

**User Story / PBI / Feature**
```html
<b>Contexto</b><p>...</p>
<b>Objetivo</b><p>...</p>
<b>Critérios de aceite</b><ul><li>...</li></ul>
```
Se o processo do projeto usar o campo `Microsoft.VSTS.Common.AcceptanceCriteria`, grave os critérios de aceite ali em vez de (ou além) da descrição.

**Bug**
```html
<b>Descrição</b><p>...</p>
<b>Passos para reproduzir</b><ol><li>...</li></ol>
<b>Comportamento esperado</b><p>...</p>
<b>Comportamento observado</b><p>...</p>
```
Se o processo do projeto usar o campo `Microsoft.VSTS.TCM.ReproSteps`, grave os passos de reprodução ali em vez de dentro da descrição.

**Task**
```html
<b>O que precisa ser feito</b><p>...</p>
<b>Definição de pronto</b><ul><li>...</li></ul>
```

Regras gerais:
- Nunca invente informação que não esteja no contexto fornecido nem nos dados já existentes no work item. Marque lacunas com `[a confirmar]` em vez de supor.
- Ao atualizar um item existente, preserve conteúdo relevante já presente na descrição — mescle em vez de substituir tudo, a menos que o usuário peça reescrita completa.

## Passo 4 — Aplicar no Azure Boards via MCP

- **Atualizar existente**: chame `wit_update_work_item` com o `id` e o patch document contendo `System.Title`, `System.Description` e demais campos definidos no Passo 3.
- **Criar novo**: chame `wit_create_work_item` com `project`, `workItemType`, `System.Title`, `System.Description` e, se confirmados com o usuário, `System.AreaPath` / `System.IterationPath`.
- Se alguma chamada ao MCP retornar erro (ex.: tipo de work item inválido para o processo do projeto, campo obrigatório ausente), reporte o erro e ajuste os dados antes de tentar novamente — não repita a mesma chamada sem corrigir a causa.

## Passo 5 — Confirmar resultado

Ao final, exiba:
- Link e ID do work item criado/atualizado
- Título final
- Descrição final (renderizada, não o HTML bruto)
- Campos marcados como `[a confirmar]` que precisam de revisão manual
