# ADR-1004 — Frontend REST-first com OpenAPI

- **Status:** Proposed
- **Data:** 2026-01-22
- **Escopo:** Frontend ↔ API Gateway

## Contexto
O frontend do Sagnus ERP precisa consumir múltiplos BCs via API Gateway,
com previsibilidade, versionamento e baixo acoplamento.

## Decisão
- Adotar **REST-first** como padrão de integração frontend ↔ backend
- APIs documentadas via **OpenAPI**
- Geração automática de client tipado (TypeScript)
- GraphQL reservado para casos futuros de composição complexa

## Diretrizes
- Frontend consome apenas o API Gateway
- DTOs gerados a partir do OpenAPI são imutáveis
- Mapeamento DTO → ViewModel feito no frontend
- Versionamento de endpoints explícito

## Alternativas consideradas
- GraphQL desde o início (descartado por custo inicial)
- Chamadas diretas aos BCs (descartado por acoplamento)

## Consequências
- Integração previsível
- Melhor DX com tipagem forte
- Disciplina maior na evolução da API
