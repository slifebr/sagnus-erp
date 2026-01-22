# ADR-1007 — Motor de Grid Forms-like

- **Status:** Accepted
- **Data:** 2026-01-22
- **Contexto:** Migração de UX Oracle Forms → Web

## Contexto
O ERP exige grids com:
- Edição inline multirow
- Copiar/colar estilo Excel
- Navegação por teclado
- Alta performance

## Decisão
- Adotar um único motor de grid encapsulado em `SagnusGrid`
- Wrapper próprio no frontend (`packages/grid`)
- Grid recomendado: AG Grid (via wrapper)

## Regras
- Telas não usam grid vendor diretamente
- Validações e estados de linha padronizados
- Clipboard TSV como padrão

## Consequências
- UX consistente
- Redução de retrabalho
- Dependência controlada do motor de grid
