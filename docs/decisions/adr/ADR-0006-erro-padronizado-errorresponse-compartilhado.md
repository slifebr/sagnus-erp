# ADR-0006 — Erro padronizado (ErrorResponse) compartilhado

## ADR-0006 — Erro padronizado (ErrorResponse) compartilhado

**Decisão:** padronizar erros em `sagnus-shared-api-error` e tratamento em `sagnus-platform-web`.

**Motivo:**
- respostas consistentes para front-end e integrações
- melhora suporte (correlationId + code por BC)

**Consequências:**
- requer convenção de códigos (ex.: `NFE-001`, `AUTH-001`)
- evita que cada BC “inventar” formato de erro
