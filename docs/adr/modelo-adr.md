# ADR-001 — Monolito modular com fatias verticais, Java e React

- **Situação:** aceita
- **Data:** 2026-10-25
- **Participantes:** Lucas Alves Batista, Kaue, Marcello

## Contexto

O Connexa é uma aplicação de grupos de estudo, desenvolvida na disciplina de Engenharia de Software. A equipe tem um único integrante e uma sprint de duas semanas, então a solução precisa ser simples de executar e de verificar. O código é escrito com apoio de um agente de IA, o que exige fronteiras de módulo explícitas e testes automatizados que comprovem cada cenário. A autenticação real está fora da Sprint 1, e o estudante é simulado no módulo de identidade. O integrante já tem conhecimento prévio em Java, o que pesou na escolha da stack.

## Decisão

Adotamos um monolito modular com fatias verticais no back-end e uma SPA no front-end, a partir do template `connexa-modelo-java`: Java 21, Spring Boot, Spring Modulith, Spring Data JPA, Flyway e PostgreSQL 17 no back-end; React 19, TypeScript e Vite no front-end. Cada módulo é dono dos seus dados, com um schema por módulo, e a API pública de um módulo é o seu pacote raiz, com o código interno em `internal/`.

## Alternativas consideradas

| Alternativa | Por que não foi escolhida |
|---|---|
| Microsserviços | Exigem infraestrutura, comunicação entre serviços e implantação separada, custo desproporcional para uma equipe de 1 pessoa em duas semanas |
| Monolito em camadas (sem módulos) | Sem fronteiras explícitas, o agente de IA pode misturar responsabilidades, e nada impede a dependência entre módulos |
| Templates `connexa-modelo-dotnet` e `connexa-modelo-node` | Equivalentes em estrutura, mas a stack Java foi escolhida por conhecimento prévio do integrante |
| Testcontainers para os testes | Exige Docker dentro do Codespace; o template usa um PostgreSQL do Compose com banco separado (`connexa_teste`), que funciona igual no Codespaces, no CI e na máquina local |

## Consequências

- **Positivas:** as fronteiras entre módulos são verificadas automaticamente a cada build; um único deploy simplifica a operação; o ambiente é reprodutível no Codespaces; cada território (grupos, encontros, painel) pode evoluir de forma independente.
- **Negativas:** o monolito escala como uma unidade só; todos os módulos compartilham o mesmo banco físico, embora em schemas separados; a disciplina das fronteiras depende de os testes e da revisão serem mantidos.
- **Como verificar:** os testes `ArquiteturaModularTest` (Spring Modulith) e `SchemasDosModulosTest` rodam em todo build; o lint do front-end bloqueia importações entre features fora do `index.ts`; o CI executa `./verificar` em todo pull request, e a revisão confere o escopo de cada mudança.
