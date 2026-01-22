# ADR-0011 — API Gateway: Agregação Read-Only

## ADR-0011 — API Gateway: Agregação Read-Only

**Decisão:** O `sagnus-api-gateway` pode fazer agregação de dados via contratos de BCs (queries read-only), mas **nunca** deve ter persistência própria ou lógica de domínio.

**Motivo:**
- Gateway é um BFF/Edge layer, não um BC
- Agregação é necessária para UX (evitar múltiplas chamadas do front)
- Persistência no Gateway duplicaria domínio e quebraria bounded contexts

**Consequências:**
- Gateway pode depender de contratos `*-api` dos BCs
- Gateway **não** pode ter tabelas próprias ou JPA repositories
- Toda lógica de negócio deve estar nos BCs
