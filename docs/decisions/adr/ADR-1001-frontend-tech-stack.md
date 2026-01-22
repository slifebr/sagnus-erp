# ADR-1001 — Tech Stack do Frontend

- **Status:** Accepted
- **Data:** 2026-01-22
- **Escopo:** Frontend (Sagnus ERP)

## Contexto

O frontend do Sagnus ERP será desenvolvido inicialmente por um único desenvolvedor,
com forte foco em produtividade, previsibilidade e capacidade de evoluir o sistema
sem retrabalho.

O sistema possui características típicas de ERP:
- telas densas de dados
- formulários extensos
- grids pesados com edição inline
- navegação intensa por teclado
- ciclo de vida longo

A stack escolhida precisa ser:
- madura
- bem suportada pelo ecossistema
- adequada para aplicações corporativas complexas
- eficiente para desenvolvimento individual

## Decisão

Adotar a seguinte stack base para o frontend:

- **React** como biblioteca principal de UI
- **TypeScript** como linguagem padrão
- **Vite** como ferramenta de build e dev server
- **React Router** para roteamento
- **ESLint + Prettier** para padronização de código

### Justificativa técnica

- React oferece grande ecossistema, flexibilidade e maturidade.
- TypeScript reduz erros em um domínio complexo (ERP).
- Vite fornece build rápido e simples, sem complexidade desnecessária.
- A stack é amplamente adotada e bem documentada, reduzindo risco de obsolescência.

## Alternativas consideradas

### Angular
- **Prós:** framework completo, padrão corporativo.
- **Contras:** maior curva de aprendizado, boilerplate elevado para desenvolvimento solo.
- **Decisão:** não adotado.

### Flutter Web
- **Prós:** UI consistente, multiplataforma.
- **Contras:** grid pesado e UX de ERP web mais complexos.
- **Decisão:** não adotado.

### Next.js
- **Prós:** SSR, SEO.
- **Contras:** SSR não é requisito para ERP interno; adiciona complexidade.
- **Decisão:** não adotado.

## Consequências

### Benefícios
- Alta produtividade
- Stack moderna e consolidada
- Facilidade de contratação futura

### Custos/Riscos
- Responsabilidade maior de definir padrões arquiteturais
- Necessidade de disciplina no uso de hooks e estado
