# ADR-0003 — Contratos `*-api` (Java) entre BCs

## ADR-0003 — Contratos `*-api` (Java) entre BCs

**Decisão:** comunicação direta em-process entre BCs (no monorepo) via contrato Java,
com possibilidade de trocar para HTTP/mensageria no futuro.

**Motivo:**
- reduz complexidade na fase inicial
- mantém acoplamento controlado e explícito (ports + DTOs mínimos)

**Consequências:**
- contratos precisam ser estáveis (versionamento, compatibilidade)
- no futuro, contrato pode ganhar implementação via HTTP sem mudar consumidor
