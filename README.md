# Sagnus ERP — Produto (Governança)

Este repositório é o **repo pai** do produto Sagnus ERP.  
Ele contém **documentação, decisões arquiteturais (ADRs), roadmap** e guias de integração.

> Importante: o código fonte fica em repositórios separados:
> - Backend: `sagnus-backend`
> - Frontend: `sagnus-frontend`

---

## Repositórios

### Backend
- Repo: **sagnus-backend**
- Local: `../sagnus/fontes/backend/sagnus`
- Responsável por: API Gateway + BCs + contratos + integrações

### Frontend
- Repo: **sagnus-frontend**
- Local: `../sagnus/fontes/frontend/sagnus-erp`
- Responsável por: Sagnus Shell + ADM/Login + UI + Grid “Forms-like”

---

## Estrutura deste repositório (governança)

- `docs/decisions/`
  - `DECISIONS.md` (índice)
  - `adr/` (1 ADR por arquivo)
- `docs/roadmap/`
- `docs/architecture/`

---

## ADRs

- Índice: `docs/decisions/DECISIONS.md`

### Frontend (numeração 1000+)
- ADR-1003 — Auth, Sessão e Tokens
- ADR-1007 — Motor de Grid “Forms-like”

---

## Como trabalhar no produto (visão geral)

1. Desenvolver backend no repo `sagnus-backend`
2. Desenvolver frontend no repo `sagnus-frontend`
3. Registrar decisões e arquitetura aqui no `sagnus-erp`

---

## Evolução futura (opcional)
Este repo poderá evoluir para usar **git submodules** apontando para backend/frontend, permitindo “congelar” versões por release do produto.
