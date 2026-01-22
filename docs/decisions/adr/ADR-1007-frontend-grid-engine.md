# ADR-1007 — Motor de Grid Forms-like

- **Status:** Accepted
- **Data:** 2026-01-22
- **Escopo:** Frontend (UX ERP)

## Contexto
A migração do Oracle Forms exige grids com:
- Edição inline multirow
- Copiar/colar estilo Excel
- Navegação intensa por teclado
- Alta performance

## Decisão
- Adotar um motor de grid único encapsulado em `SagnusGrid`
- Wrapper próprio no frontend (`packages/grid`)
- Motor recomendado: AG Grid (isolado via wrapper)

## Consequências
- UX consistente
- Redução de retrabalho
- Dependência de vendor controlada
