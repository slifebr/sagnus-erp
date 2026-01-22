# ADR-1002 — Estrutura de Repositório do Frontend

- **Status:** Accepted
- **Data:** 2026-01-22
- **Escopo:** Frontend (Organização de Código)

## Contexto

O Sagnus ERP é um produto composto por múltiplos projetos:
- backend
- frontend
- documentação e governança

Cada parte possui ciclos de vida e preocupações distintas.
Misturar frontend e backend no mesmo repositório tende a gerar:
- histórico confuso
- pipelines complexos
- acoplamento indevido

Além disso, o frontend crescerá em módulos (ADM, Corp, Financeiro, etc.)
e precisará compartilhar código (UI, contracts, grid, core).

## Decisão

Adotar **repositórios separados**, com um **repositório pai de governança**:

### Repositórios
- `sagnus-backend` → código backend
- `sagnus-frontend` → código frontend
- `sagnus-erp` → documentação, ADRs, arquitetura, roadmap

### Organização interna do frontend
O repositório `sagnus-frontend` utilizará **monorepo com workspaces**:

- `apps/`
  - aplicação principal (Sagnus Shell)
- `packages/`
  - `ui` (design system)
  - `core` (http, auth, errors)
  - `contracts` (DTOs gerados do OpenAPI)
  - `grid` (motor Forms-like)

Ferramenta de workspaces: **pnpm** (preferencial), com npm/yarn como alternativas.

## Alternativas consideradas

### Repositório único (backend + frontend)
- **Prós:** tudo em um lugar.
- **Contras:** forte acoplamento, governança ruim.
- **Decisão:** não adotado.

### Repositórios totalmente independentes sem monorepo
- **Prós:** simplicidade inicial.
- **Contras:** duplicação de código e inconsistência.
- **Decisão:** não adotado.

## Consequências

### Benefícios
- Separação clara de responsabilidades
- Governança forte do produto
- Reuso de código controlado
- Evolução independente de frontend e backend

### Custos/Riscos
- Requer disciplina de versionamento interno
- Leve aumento de complexidade inicial no setup
