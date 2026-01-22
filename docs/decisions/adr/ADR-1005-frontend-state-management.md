# ADR-1005 — State Management no Frontend

- **Status:** Accepted
- **Data:** 2026-01-22
- **Escopo:** Frontend (Estado de Aplicação)

## Contexto
O frontend do Sagnus ERP lida com:
- Dados remotos (server-state)
- Estado de UI (layout, tabs, foco)
- Sessão e preferências do usuário

Misturar esses tipos de estado gera acoplamento e bugs difíceis de rastrear.

## Decisão
- **TanStack Query** para server-state:
  - cache
  - refetch
  - paginação
  - sincronização com backend
- **Zustand** para client/UI-state:
  - sessão autenticada
  - tabs abertas
  - preferências do usuário
- Evitar Redux como padrão (permitido apenas com ADR específico)

## Diretrizes
- Nenhuma chamada HTTP direta fora do client central
- Estado global mínimo e explícito
- Queries nomeadas e versionadas

## Consequências
- Separação clara de responsabilidades
- Melhor performance e previsibilidade
- Redução de boilerplate
