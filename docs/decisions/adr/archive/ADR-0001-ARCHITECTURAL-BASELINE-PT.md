# ADR-0001 – Baseline Arquitetural do Sagnus ERP

## Status
ACCEPTED

## Data
2026-01-19

## Contexto
O Sagnus ERP é a modernização de um ERP legado baseado em Oracle Forms 6i,
com forte dependência de banco Oracle e lógica distribuída entre Forms e PL/SQL.

O objetivo é evoluir para uma plataforma Web moderna, sustentável e de longo prazo,
com independência tecnológica e clareza arquitetural.

## Decisão
A arquitetura oficial do Sagnus ERP passa a ser definida pelos seguintes princípios:

- DDD (Domain-Driven Design) + Arquitetura Hexagonal
- Monorepo Maven multi-módulo
- Um Bounded Context por módulo
- Domínio implementado em Java puro
  - Sem Spring
  - Sem JPA
  - Sem frameworks
- Repositórios definidos como PORTS no domínio
- Application Layer responsável por orquestrar casos de uso
- Infrastructure Layer responsável por ADAPTERS técnicos
- Controllers sem lógica de negócio
- Gateway atuando como BFF fino
- Estratégia GraphQL-first
- REST apenas quando houver justificativa técnica
- Contracts isolados em módulos `sagnus-bc-contracts-*`
- Banco de dados agnóstico (Oracle e PostgreSQL)

## Consequências

### Passa a ser obrigatório
- Respeitar ports & adapters
- Manter o domínio puro
- Evitar acoplamento direto entre BCs
- Versionar contratos quando necessário

### Passa a exigir novo ADR
- Criação ou divisão de Bounded Context
- Introdução de exceções ao domínio puro
- Mudança na estratégia de Gateway ou API
- Introdução de novos padrões transversais

## Observações
Este ADR define o estado base da arquitetura.
Toda evolução futura deve partir deste baseline.

## Referências
- .ai-context/
- docs/CHECKLIST_MERGE.md
- docs/CONVENTIONAL_COMMITS.md
