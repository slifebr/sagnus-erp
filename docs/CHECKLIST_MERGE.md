# Sagnus ERP – Merge / PR Checklist

Este checklist é obrigatório para PRs, merges locais e hotfixes.
Ele existe para garantir consistência DDD + Hexagonal e evitar regressões arquiteturais.

---

## 1) Arquitetura (DDD + Hexagonal)
- [ ] Domínio permanece **Java puro** (sem Spring/JPA/anotações/frameworks)
- [ ] Nenhuma entidade JPA foi movida/duplicada para o domínio
- [ ] Repositórios no domínio são **PORTS** (interfaces)
- [ ] Infraestrutura contém apenas **ADAPTERS** (implementações técnicas)
- [ ] Application orquestra casos de uso (não “vaza” regra para controller/infra)
- [ ] Controllers **não contêm lógica de negócio** (somente I/O e validações simples)

## 2) Módulos e limites (Bounded Contexts)
- [ ] Um Bounded Context por módulo
- [ ] Não foi criado `bc-*-api` sem justificativa explícita e documentada (ADR)
- [ ] Contracts estão em `sagnus-bc-contracts-*`
- [ ] Não existem dependências circulares entre BCs
- [ ] Integrações entre BCs usam contracts/ports (não “import direto” de domínio alheio)

## 3) Código (stack e padrões)
- [ ] Java 21 + Spring Boot 3.x
- [ ] Lombok usado de forma consistente (evitar boilerplate manual)
- [ ] MapStruct usado para mapeamentos (evitar mapper manual disperso)
- [ ] Tratamento de exceções e erros está centralizado (application / gateway)
- [ ] Logs e observabilidade seguem padrão do projeto

## 4) API / Gateway / GraphQL
- [ ] Gateway continua **BFF fino** (não virou backend “centralizador”)
- [ ] GraphQL-first (REST apenas se houver justificativa)
- [ ] Contracts e schemas foram versionados quando necessário

## 5) Banco de dados (agnóstico)
- [ ] Código permanece DB agnóstico (Oracle/PostgreSQL)
- [ ] Nenhum `quoted identifier` foi introduzido
- [ ] Nomenclatura em português conforme convenções do projeto
- [ ] Migrações versionadas (quando aplicável) e idempotência verificada

## 6) Testes e segurança
- [ ] Testes adicionados/ajustados quando existe regra de negócio ou mapeamento sensível
- [ ] Nenhum segredo (senha/token/chave) foi commitado
- [ ] Validações e autorização/autenticação não foram enfraquecidas

## 7) Documentação e governança
- [ ] Se houve mudança arquitetural/limites/nomeação/camadas → **ADR criado**
- [ ] Alterações descrevem classes sempre como **package + Classe**
- [ ] README/Docs atualizados quando a mudança afeta onboarding ou uso

---

## Padrão de mensagem de commit (Conventional Commits)
Use o guia em: `docs/CONVENTIONAL_COMMITS.md`

Sugestão para mudanças de governança/config:
- `chore(governance): ...`
- `chore(ai-context): ...`
- `docs(adr): ...` (apenas para texto de ADR sem mudança estrutural adicional)
