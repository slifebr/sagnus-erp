# ADR-0009 — Roadmap NFe: Domínio → Application → Infra (SEFAZ/XML)

## ADR-0009 — Roadmap NFe: Domínio → Application → Infra (SEFAZ/XML)

**Decisão:** construir o BC NFe na sequência:
1) Domínio puro
2) UseCases + Ports
3) Infra (persistência)
4) Integração SEFAZ / XML / eventos

**Motivo:**
- congela regras fiscais antes de “espalhar” para fora
- evita refactor caro depois

**Consequências:**
- entrega inicial sem SEFAZ, mas com base sólida
- permite testes unitários desde o início
