---
agent: agent
description: Analisa as alterações da branch atual, monta título e descrição do pull request e cria no Azure DevOps após confirmação
---

Você tem acesso ao terminal (git) e ao servidor MCP do Azure DevOps. Use o git localmente para analisar commits e diffs, e o MCP do Azure DevOps (ex.: `repo_create_pull_request`, `repo_list_repos_by_project`, `wit_link_work_item_to_pull_request`) para criar o pull request e vincular work items.

**Nunca crie o pull request sem confirmação explícita do usuário** (Passo 5 é um ponto de parada obrigatório).

## Passo 1 — Identificar branch atual, branch base e repositório

- Branch atual: `git branch --show-current`
- Branch base: pergunte ao usuário se não for óbvia pelo contexto (padrão comum: `main` ou `master`). Verifique com `git merge-base --fork-point` ou `git log --oneline <base>..HEAD` se a branch base informada realmente é ancestral.
- Repositório e projeto no Azure DevOps: se não estiverem claros, use `repo_list_repos_by_project` (e `core_list_projects` se o projeto também for ambíguo) para confirmar com o usuário.
- Confirme que a branch atual está publicada no remoto e atualizada (`git status`, `git log origin/<branch>..HEAD`). Se houver commits locais não enviados, avise o usuário e pergunte se deseja que você faça `git push` antes de prosseguir — não empurre a branch sem essa confirmação.

## Passo 2 — Listar commits e arquivos alterados

- Commits da branch: `git log <base>..HEAD --format="%h %s"`
- Para cada commit, arquivos alterados: `git show --stat --format="" <hash>`
- Arquivos alterados no total (agregado): `git diff <base>...HEAD --stat`
- Diff completo para leitura de contexto: `git diff <base>...HEAD`

## Passo 3 — Analisar o contexto das alterações

Leia os diffs e entenda o propósito de cada mudança (não apenas o que mudou, mas por quê — infira pela própria alteração de código, nomes de arquivos e mensagens de commit).

- **Se houver um único commit**: monte uma descrição única e coesa do que foi alterado, sem necessidade de detalhar por commit.
- **Se houver mais de um commit**: monte uma lista onde cada item traz o hash curto do commit seguido do contexto/motivo daquela alteração específica, por exemplo:
  ```
  - `a1b2c3d` — Corrige validação de CPF que aceitava dígitos verificadores inválidos
  - `e4f5a6b` — Adiciona testes de borda para CPFs com todos os dígitos iguais
  ```
  Não copie a mensagem de commit literalmente se ela for vaga (ex.: "fix", "wip", "ajustes") — nesse caso, descreva o que o diff daquele commit realmente faz.

## Passo 4 — Detectar work items relacionados (Azure Boards)

- Procure por referências a work items nas mensagens de commit e no nome da branch (padrões como `#1234`, `AB#1234`, `1234` em branches do tipo `feature/1234-...` ou `bugfix/1234-...`).
- Se encontrar, liste os IDs para vincular ao pull request no Passo 6.
- Se não encontrar nenhuma referência, siga sem work item vinculado — não invente um ID.

## Passo 5 — Gerar título e descrição do pull request

**Título** (`System.Title` do PR): conciso, no imperativo, sem ponto final, resumindo a mudança principal da branch como um todo (não apenas o último commit).

**Descrição**, estruturada assim:

```markdown
## Resumo
<resumo geral do que a branch entrega, 1-3 frases>

## Alterações
<lista por commit (Passo 3), ou descrição única se houver apenas um commit>

## Arquivos alterados
<lista dos arquivos alterados, agrupados por tipo de mudança: adicionado/modificado/removido>

## Work items relacionados
AB#<id> (se algum tiver sido detectado no Passo 4; omitir esta seção se não houver nenhum)
```

## Passo 6 — Apresentar para confirmação (obrigatório)

Exiba o título e a descrição completos gerados no Passo 5 e pergunte explicitamente ao usuário se pode criar o pull request com esse conteúdo. Se o usuário pedir ajustes, refaça o texto e apresente novamente — só avance para o Passo 7 após confirmação explícita ("sim", "pode criar", "confirmado" ou equivalente).

## Passo 7 — Criar o pull request via MCP

- Chame `repo_create_pull_request` com `repositoryId`, branch de origem, branch de destino, título e descrição confirmados no Passo 6.
- Se houver work items detectados no Passo 4, vincule-os com `wit_link_work_item_to_pull_request` (a menção `AB#<id>` na descrição já cria o link automático no Azure Boards, mas reforce com a ferramenta de vínculo quando disponível).
- Se a chamada ao MCP retornar erro (branch de origem não publicada, PR já existente para essas branches, permissão insuficiente etc.), reporte o erro ao usuário e não tente novamente sem corrigir a causa.

## Passo 8 — Confirmar resultado

Ao final, exiba:
- Link e ID do pull request criado
- Branch de origem e destino
- Work items vinculados (se houver)
