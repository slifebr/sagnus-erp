# ADR-0001 — Monorepo Maven multi-módulo

## ADR-0001 — Monorepo Maven multi-módulo

**Decisão:** usar Maven multi-módulo em um único repositório.

**Motivo:**
- build consistente e reprodutível
- contratos `*-api` versionados no mesmo ciclo
- facilita refactor inicial e acelera evolução do “core”

**Consequências:**
- IDE pode exigir reload/reopen para reindexar módulos
- versionamento de releases pode precisar de disciplina (tags/releases por marco)
