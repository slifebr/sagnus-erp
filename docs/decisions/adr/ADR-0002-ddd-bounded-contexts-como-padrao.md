# ADR-0002 — DDD + Bounded Contexts como padrão

## ADR-0002 — DDD + Bounded Contexts como padrão

**Decisão:** modelar o ERP por Bounded Contexts (AUTH, CORP, NFe…).

**Motivo:**
- ERP fiscal cresce e muda muito: separar contextos reduz risco
- linguagens de domínio diferentes por área (cadastro vs fiscal)
- equipes podem evoluir módulos sem travar o todo

**Consequências:**
- exige contratos explícitos e governança de dependências
- evita “atalhos” (importar entidade/JPA do vizinho)
