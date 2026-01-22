# ADR-1004 — Frontend REST-first com OpenAPI

- **Status:** Accepted
- **Data:** 2026-01-22
- **Escopo:** Frontend ↔ API Gateway

## Contexto
O frontend do Sagnus ERP consome múltiplos Bounded Contexts via API Gateway.
É necessário previsibilidade, tipagem forte e versionamento explícito.

## Decisão
- Adotar **REST-first** como padrão de integração frontend ↔ backend
- APIs documentadas via **OpenAPI**
- Geração automática de client tipado (TypeScript)
- Frontend consome **exclusivamente** o API Gateway
- GraphQL reservado para cenários futuros de composição complexa

## Diretrizes
- DTOs gerados do OpenAPI são imutáveis
- Conversão DTO → ViewModel ocorre no frontend
- Versionamento explícito de endpoints

## Consequências
- Integração previsível
- Melhor DX com tipagem forte
- Evolução controlada da API
