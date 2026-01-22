# ADR-0008 — Segredos por variáveis de ambiente

## ADR-0008 — Segredos por variáveis de ambiente

**Decisão:** usar env vars para DB/JWT secrets, não commitar senhas em YAML.

**Motivo:**
- segurança e compliance
- compatível com CI/CD e ambientes múltiplos

**Consequências:**
- dev precisa configurar ambiente local
- mais tarde podemos integrar Vault/KMS/Secrets do CI
