# ADR-0005 — Segurança como plataforma (JWT stateless)

## ADR-0005 — Segurança como plataforma (JWT stateless)

**Decisão:** consolidar segurança em `sagnus-platform-security`.

**Motivo:**
- padroniza filtros, configuração e claims em todos os BCs
- reduz duplicação e divergências de segurança

**Consequências:**
- mudanças de segurança impactam todos os BCs (mas de forma controlada)
- exige integração com `platform-web` para padronizar 401/403
