# ADR-0004 — Domínio puro (sem framework)

## ADR-0004 — Domínio puro (sem framework)

**Decisão:** o domínio não depende de Spring/JPA/HTTP/JWT.

**Motivo:**
- domínio altamente testável
- regras fiscais ficam “blindadas” contra mudanças de infra
- facilita portabilidade e reuso

**Consequências:**
- mapeamentos para persistência/XML ficam em infra/adapters
- exige disciplina (não “vazar” anotações no domínio)
