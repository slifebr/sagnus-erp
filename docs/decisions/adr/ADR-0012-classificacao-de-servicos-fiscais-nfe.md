# ADR-0012 — Classificação de Serviços Fiscais (NFe)

## ADR-0012 — Classificação de Serviços Fiscais (NFe)

**Decisão:** Regras fiscais (IVA Dual IBS/CBS) devem estar centralizadas em **Domain Services** dentro do BC NFe:
- `CalculadoraIvaService` → `domain/service/`
- Orquestração de emissão → `application/usecase/EmitirNfeUseCase`
- Integração SEFAZ → `infrastructure/http/SefazClient`

**Motivo:**
- Cálculos fiscais são **regras de domínio puras** (não dependem de infra)
- Centralização evita duplicação e inconsistências
- Facilita testes unitários (sem Spring)

**Consequências:**
- `CalculadoraIvaService` não pode depender de Spring/JPA
- Use cases chamam domain services para cálculos
- Adapters (XML, SEFAZ) ficam em infrastructure
