# Sagnus ERP – Conventional Commits (Padrão Oficial)

Este projeto adota **Conventional Commits** para:
- padronizar histórico,
- melhorar revisão,
- habilitar changelog/semver automatizados (quando aplicável),
- separar claramente mudanças de produto vs manutenção.

Formato:

```
<type>(<scope>): <subject>
```

Exemplo:
```
feat(corp): add createPessoa use case with validation
fix(fina): handle null tax rate in IVA calculation
chore(ai-context): add canonical PT + EN AI context
```

---

## 1) Types (tipos permitidos e recomendados)

### ✅ Tipos principais (uso comum)
- **feat**: nova funcionalidade (impacta comportamento do sistema)
- **fix**: correção de bug (comportamento incorreto)
- **refactor**: refatoração sem mudar comportamento (reorganização de código)
- **perf**: melhoria de performance (pode alterar comportamento de tempo/recursos)
- **test**: inclusão/ajuste de testes
- **docs**: documentação (README, ADRs, guias) sem mudança funcional
- **build**: mudanças em build (Maven, plugins, versionamento)
- **ci**: pipelines (GitHub Actions, Jenkins, etc.)
- **chore**: manutenção/infra/arquivos de suporte (sem feature/bugfix)
- **style**: formatação (sem mudança de lógica) — usar com moderação

### ❌ Importante
- **`core` não é um type oficial** no padrão Conventional Commits.
  - Use `chore` ou um `scope` adequado (ex.: `chore(core): ...` se fizer sentido).

---

## 2) Scopes (escopos recomendados no Sagnus)

Use escopo para indicar o BC/módulo/área afetada.

### Bounded Contexts (exemplos)
- `corp`, `adm`, `fina`, `estoque`, `nfe`, `gateway`

### Governança / arquitetura / tooling
- `ai-context`, `governance`, `adr`, `contracts`, `scripts`, `build`, `ci`

Exemplos:
- `chore(governance): add merge checklist and commit guide`
- `docs(adr): add ADR-0013 for gateway graphql strategy`
- `chore(contracts): publish v2 of corp contracts`
- `build(parent): update maven plugin versions`

---

## 3) Subject (assunto) – regras de escrita
- Frase curta, objetiva, no imperativo (ex.: “add”, “fix”, “remove”, “update”)
- Sem ponto final
- Evite detalhes demais (detalhes vão no corpo)

Bom:
- `feat(corp): add pessoa search with pagination`
Ruim:
- `feat: I added a lot of stuff and changed many files.`

---

## 4) Corpo do commit (quando usar)
Use o corpo quando:
- houver impacto arquitetural,
- houver migração,
- houver risco,
- houver breaking change,
- o PR for grande.

Exemplo:

```
refactor(fina): isolate tax calculation ports

- Move tax calculation orchestration into application layer
- Keep domain pure (no Spring/JPA)
- Update adapters in infrastructure

Refs: fina.application.usecase.CalcImpostosUseCase
```

---

## 5) Breaking Changes
Quando a mudança for incompatível:
- inclua `!` após o type/scope:
  - `feat(gateway)!: rename graphql endpoint`
- e descreva no corpo com `BREAKING CHANGE:`

---

## 6) Regras práticas (para o dia a dia)

### Quando usar `chore`
Use **chore** para mudanças que não são feature nem bugfix, por exemplo:
- `.ai-context/`, `.agent/`, `.cursorrules`
- checklists, templates, scripts
- ajustes de build/pastas/infra
- atualização de dependências sem mudança funcional

Exemplos:
- `chore(ai-context): add canonical PT + EN mirrored context`
- `chore(scripts): update new-bc-from-sql to create graphql folder per BC`
- `chore(governance): add PR checklist and ADR template`

### Quando usar `docs`
Use **docs** para documentação pura (sem mudança estrutural):
- `docs(adr): add ADR-0012 for contracts module naming`
- `docs(readme): clarify local setup steps`

---

## 7) Padrão recomendado para mudanças de arquitetura
Se você estiver mudando:
- limites de BC,
- camadas (domain/application/infrastructure),
- estratégia de contracts,
- gateway/graphql,

então:
1) crie/atualize ADR,
2) use `refactor`/`chore` conforme o caso,
3) cite classes como **package + Classe** no corpo do commit.
