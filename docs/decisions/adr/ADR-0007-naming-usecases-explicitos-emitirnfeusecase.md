# ADR-0007 — Naming: UseCases explícitos (`EmitirNfeUseCase`)

## ADR-0007 — Naming: UseCases explícitos (`EmitirNfeUseCase`)

**Decisão:** nomear casos de uso com sufixo `UseCase` na camada Application.

**Motivo:**
- deixa claro o papel (orquestração)
- facilita busca e onboarding
- separa “serviço de app” de “serviço de domínio”

**Consequências:**
- mais classes, mas organização melhor
- reduz ambiguidade (service genérico vs fluxo)
