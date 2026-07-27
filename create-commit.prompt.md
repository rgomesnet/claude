---
agent: agent
description: Analisa alterações em staged e não staged, decide o que entra no commit e gera a mensagem com base no contexto das mudanças
---

Você tem acesso ao terminal (git). Use-o para analisar o estado do repositório antes de decidir o que será commitado.

**Nunca crie o commit sem confirmação explícita do usuário** (Passo 5 é um ponto de parada obrigatório). Nunca use `git add -A` ou `git add .` — adicione sempre arquivos específicos pelo nome.

## Passo 1 — Levantar o estado do repositório

- `git status --short` para ver, de uma vez, o que está em staged, o que foi modificado mas não está em staged, e o que é untracked.
- `git diff --cached --stat` → alterações já em staged.
- `git diff --stat` → alterações em arquivos rastreados, ainda não staged.
- Untracked files → listados no `git status`.

## Passo 2 — Decidir o que entra no commit

- Arquivos já em staged: por padrão, fazem parte do commit.
- Arquivos modificados fora da staged area: **pergunte ao usuário, um a um ou em grupo, se deseja incluí-los no commit.** Não presuma que sim.
- Arquivos untracked: mesma regra — pergunte antes de incluir. Se o nome do arquivo sugerir segredo/credencial (`.env`, `*secret*`, `*credentials*`, chaves privadas, etc.), leia o conteúdo antes de sugerir a inclusão e alerte o usuário explicitamente, mesmo que ele já tenha pedido para incluí-lo.
- Monte a lista final de arquivos confirmados antes de prosseguir.

## Passo 3 — Analisar o contexto das alterações

- Rode `git diff --cached` (após incluir no staging os arquivos confirmados no Passo 2) para ler o diff completo do que será commitado.
- Rode `git log --oneline -10` para seguir o estilo de mensagens já usado neste repositório (idioma, tom, nível de detalhe).
- Classifique a natureza da mudança: novo arquivo/funcionalidade, atualização de conteúdo existente, correção, refatoração, etc. — isso define o verbo usado na mensagem.
- Entenda o "porquê" da mudança a partir do próprio diff (não apenas liste o que mudou).

## Passo 4 — Gerar a mensagem de commit

- Primeira linha: resumo direto, no imperativo, sem ponto final, seguindo o idioma e o tom dos commits recentes do repositório (Passo 3).
- Corpo (opcional): inclua apenas se a mudança não for óbvia pela primeira linha ou envolver múltiplos arquivos com propósitos distintos — explique o porquê, não o que (o diff já mostra o que mudou).
- Não invente motivação que não esteja evidente no diff ou no contexto da conversa.

## Passo 5 — Apresentar para confirmação (obrigatório)

Exiba:
- Lista final de arquivos que serão incluídos no commit (já staged + recém-confirmados no Passo 2)
- Mensagem de commit gerada no Passo 4

Pergunte explicitamente se pode prosseguir. Se o usuário pedir ajustes na mensagem ou na lista de arquivos, refaça e apresente novamente — só avance para o Passo 6 após confirmação explícita.

## Passo 6 — Executar o commit

- `git add <arquivo1> <arquivo2> ...` apenas para os arquivos confirmados que ainda não estavam em staged.
- `git commit -m "<mensagem confirmada>"`.
- `git status` para confirmar que o commit foi criado e que nada ficou pendente sem intenção.

## Passo 7 — Confirmar resultado

Ao final, exiba:
- Hash curto do commit criado
- Lista de arquivos incluídos
- Mensagem final do commit
