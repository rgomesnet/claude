---
name: generate-tests
description: Gera testes unitários completos para uma classe C# em qualquer solução .NET
---

Gere testes unitários completos para a classe $ARGUMENTS seguindo essas regras:

## Descoberta automática do projeto

1. Leia o arquivo `.sln` na raiz para identificar todos os projetos da solução
2. Identifique automaticamente qual é o projeto de testes (procure por projetos
   com sufixo `.Tests`, `.UnitTests` ou `.Specs` no nome)
3. Descubra o namespace correto lendo o arquivo da classe informada
4. Nunca assuma nomes fixos de projeto ou namespace

## Regras de geração dos testes

5. Use xUnit como framework de testes
6. Siga rigorosamente o padrão Arrange/Act/Assert com comentários separando as seções
7. Nomeie os testes em inglês no padrão:
   `MetodoTestado_Cenario_ResultadoEsperado`

## Cobertura obrigatória

8. Cubra todos os cenários felizes (happy path)
9. Cubra todos os casos de erro e exceções esperadas
10. Valide casos de borda:
    - Strings nulas ou vazias
    - Números negativos ou zero
    - Listas vazias ou nulas
    - Estados inválidos de objetos
11. Verifique se existem regras de negócio implícitas no código e gere
    testes para elas também

## Qualidade dos testes

12. Cada teste deve testar uma única coisa
13. Não duplique testes com a mesma intenção
14. Use `FluentAssertions` se o projeto já o tiver como dependência,
    caso contrário use `Assert` do xUnit
15. Para dependências externas, use mocks com `Moq` se disponível na solução

## Após gerar os testes

16. Execute `dotnet test` para validar que todos os testes compilam e passam
17. Se algum teste falhar por falta de validação na classe original,
    informe quais validações estão faltando na implementação
18. Apresente um resumo com:
    - Total de testes gerados
    - Quantos são happy path
    - Quantos são casos de erro
    - Quantos são casos de borda
    - Se há alguma regra de negócio não coberta por testes