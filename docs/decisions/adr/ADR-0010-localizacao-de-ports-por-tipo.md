# ADR-0010 — Localização de Ports por Tipo

## ADR-0010 — Localização de Ports por Tipo

**Decisão:** Ports devem ser organizados por tipo e responsabilidade:
- **Repositórios:** `domain/repository/` (contratos de persistência do domínio)
- **Integrações externas:** `application/port/out/` (SEFAZ, email, APIs externas)
- **Casos de uso (opcional):** `application/port/in/` (se necessário explicitar contratos de entrada)

**Motivo:**
- Repositórios são conceitos do domínio (agregados precisam ser persistidos)
- Integrações externas são detalhes de aplicação (não fazem parte do core domain)
- Separação clara facilita testes e manutenção

**Consequências:**
- Ports de repositório sempre ficam no domínio
- Ports de integração (HTTP, messaging, etc.) ficam em application
- Reduz ambiguidade sobre onde criar novos ports
