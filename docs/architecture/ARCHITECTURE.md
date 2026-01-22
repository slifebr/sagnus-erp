# Sagnus ERP — Architecture

Data: 2025-12-16

Este documento descreve a arquitetura do **Sagnus ERP** no nível de repositório, módulos e padrões
adotados (DDD, Bounded Contexts, contratos entre contextos, camadas, segurança e erros).

> **Regra #1:** Domínio antes de framework.  
> **Regra #2:** Um Bounded Context não acessa o banco do outro.  
> **Regra #3:** Integrações entre BCs são **contratos** (API Java) ou **mensageria/HTTP** — nunca “importar entidade/JPA do vizinho”.

---

## 1. Visão geral

O Sagnus ERP é um ERP modular desenhado para suportar evolução constante (especialmente fiscal),
com **isolamento de contextos** e **acoplamento controlado**.

Arquitetura base:

- **DDD (Domain-Driven Design)**
- **Bounded Contexts (BCs)**
- **Arquitetura Limpa / Hexagonal (Ports & Adapters)**
- **Domínio puro (POJOs/VOs/Entities)**
- **Camadas: Domain → Application → Infrastructure → API**
- **JWT stateless via módulo de plataforma**
- **Erro padronizado compartilhado (ErrorResponse)**
- **Contratos explícitos entre BCs (módulos `*-api`)**

---

## 2. Layout do repositório (macro)

```
sagnus
├─ sagnus-shared-api-error         # Contratos de erro e exceções base (reutilizável)
├─ sagnus-platform-web             # Infra web comum (handlers, filtros, correlação, etc.)
├─ sagnus-platform-security        # Infra de segurança comum (JWT, filtros, config)
│
├─ sagnus-bc-corp                  # BC CORP (cadastros centrais)
├─ sagnus-bc-corp-contracts              # Contratos CORP (ports + DTOs estáveis)
│
├─ sagnus-bc-auth                  # BC AUTH (autenticação/autorização)
├─ sagnus-bc-nfe                   # BC NFe (fiscal)
│
├─ pom.xml                         # Maven multi-módulo (parent)
├─ README.md
└─ ARCHITECTURE.md                 # Este documento
```

---

## 3. Bounded Contexts

### 3.1 CORP (Cadastros centrais)
**Responsabilidade:** dados corporativos compartilháveis (Pessoa/Produto no início).

- Exemplo: Pessoa (Física/Jurídica), endereços e derivados
- CORP é a “fonte de verdade” para dados usados por AUTH e NFe

**Contratos:** `sagnus-bc-corp-contracts` expõe portas e DTOs mínimos e estáveis, por exemplo:
- `CorpPessoaQueryPort`
- `PessoaResumoDTO` (nome, documento, tipo… apenas o necessário)

### 3.2 AUTH (Autenticação/Autorização)
**Responsabilidade:** login, emissão/validação de tokens, usuários do sistema (AUTH_USUARIO).

- AUTH **não conhece** entidades/tabelas do CORP
- AUTH usa CORP apenas via contrato (`sagnus-bc-corp-contracts`) para:
  - validar `pessoa_id`
  - montar “resumo do usuário logado” com dados básicos da pessoa

### 3.3 NFe (Fiscal)
**Responsabilidade:** regras fiscais e ciclo de vida da NF-e (domínio fiscal).

- Domínio puro: `Nfe`, `NfeItem`, `NfeTotais`, VOs (Dinheiro/Quantidade/DocumentoFiscal), tributos, estados.
- Application: casos de uso (ex.: `EmitirNfeUseCase`) orquestrando domínio + portas.
- Infra: persistência, integração SEFAZ, geração XML (etapas futuras).

---

## 4. Camadas internas de um BC

A organização padrão dentro de um BC segue:

### 4.1 Domain (puro)
- Entities, Value Objects, Aggregates
- Regras/invariantes de negócio
- Exceções de domínio
- **Sem Spring/JPA/HTTP/JWT**

### 4.2 Application (orquestração)
- UseCases / AppServices
- Commands/Results
- Ports (interfaces) para dependências externas (DB, CORP, SEFAZ)
- Transação (quando necessário) tende a ficar aqui/infra, mas nunca no domínio

### 4.3 Infrastructure (adapters)
- Implementações das Ports:
  - JPA repositories
  - clients HTTP/Feing
  - adapters para mensageria
- Mappers (domínio ↔ persistência ↔ DTO)

### 4.4 API (Controllers)
- Controllers REST
- DTOs de request/response
- Validação de entrada (`@Valid`)
- Sem regra de negócio (isso fica em Application/Domain)

---

## 5. Regras de dependência (importante)

### 5.1 Dependências permitidas (regra prática)
- `bc-auth` → pode depender de `bc-corp-contracts` (contrato), **não** de `bc-corp`
- `bc-nfe` → pode depender de `bc-corp-contracts` (contrato), **não** de `bc-corp`
- BCs podem depender de módulos de plataforma:
  - `sagnus-shared-api-error`
  - `sagnus-platform-web`
  - `sagnus-platform-security`

### 5.2 Proibido
- `bc-nfe` importar `com.slifesys.sagnus.corp.domain...`
- `bc-auth` acessar tabelas do CORP
- Compartilhar “Entity JPA” entre BCs
- “Bypass” de contrato via reuso de repositório/mapper de outro BC

---

## 6. Contratos entre BCs

### 6.1 O que é contrato?
Contrato é um módulo `*-api` contendo:
- **Ports** (interfaces) que descrevem capacidades expostas
- **DTOs** estáveis e minimalistas
- (Opcional) enums e tipos de apoio

Exemplo de uso:
- AUTH recebe um `pessoa_id`
- AUTH chama `CorpPessoaQueryPort.obterResumoPorId(pessoaId)`
- AUTH não precisa conhecer a modelagem interna do CORP

### 6.2 Benefícios
- isolamento do domínio
- evolução independente
- troca futura de integração (Java in-process ↔ HTTP ↔ mensageria) sem quebrar o BC consumidor

---

## 7. Segurança (JWT stateless)

### 7.1 Onde fica
- `sagnus-platform-security`: filtros, config base, utilitários JWT (padrão do ERP)

### 7.2 Como os BCs usam
- BCs recebem requisições com `Authorization: Bearer <token>`
- Controllers/UseCases leem claims/roles via `Authentication` (Spring Security)
- Regras de autorização via `@PreAuthorize` / authorities

### 7.3 Tokens
- Access Token: curta duração
- Refresh Token: renovação
- Evolução prevista: `tokenVersion` / invalidar tokens antigos / auditoria

---

## 8. Erros e exceções (padrão ERP)

### 8.1 Onde fica
- `sagnus-shared-api-error`: contratos de erro reutilizáveis
  - `ErrorType`
  - `ErrorResponse` (com `fieldErrors`, `correlationId`, `timestamp`)
  - Exceções base: `BusinessException`, `NotFoundException`, `IntegrationException`

### 8.2 Onde trata
- `sagnus-platform-web`: `GlobalExceptionHandler` (ou handlers base)
- BCs podem ter handlers específicos **somente** para enriquecer detalhes, mantendo padrão

### 8.3 Code padrão por BC
Recomendação:
- `CORP-001`, `CORP-002`...
- `AUTH-001`, `AUTH-002`...
- `NFE-001`, `NFE-002`...

Isso facilita suporte e rastreio.

---

## 9. Persistência e banco

Regra por BC:
- um BC é dono do seu schema/tabelas
- repositórios JPA do BC só mapeiam tabelas do próprio BC

Migração de schema:
- recomendado usar Flyway/Liquibase por módulo (etapa futura)
- ou scripts versionados por BC

---

## 10. Build (Maven multi-módulo)

- `pom.xml` raiz agrega módulos
- Cada módulo tem seu `pom.xml` próprio
- Versionamento: `1.0.0-SNAPSHOT` durante construção

Comandos úteis:
- Compilar tudo:
  - `mvn clean install`
- Compilar um módulo e dependências:
  - `mvn -pl sagnus-bc-auth -am clean test -DskipTests`

---

## 11. Testes

### 11.1 Domínio
- testes unitários puros (JUnit), sem Spring
- valida invariantes: CPF/CNPJ, cálculo de totais, estados do agregado

### 11.2 Application
- mocks das Ports (NfeRepository, CorpPessoaGatewayPort etc.)
- valida orquestração e regras de fluxo

### 11.3 Infra
- testes de integração (SpringBootTest/Testcontainers) — etapa futura

---

## 12. Roadmap arquitetural (próximos marcos)

### NFe
- Infra JPA (persistência do agregado)
- Geração XML NF-e
- Integração SEFAZ (autorização, consulta, eventos)
- Eventos fiscais + mensageria (Outbox/Saga se necessário)

### Plataforma
- CorrelationId end-to-end
- Auditoria (quem fez o quê, quando)
- Observabilidade (logs estruturados, métricas)

---

## 13. Convenções recomendadas

### 13.1 Nomes
- UseCases: `EmitirNfeUseCase`, `ObterPessoaUseCase`, etc.
- Ports: `XxxRepository`, `XxxGatewayPort`, `XxxClientPort`
- Adapters: `XxxRepositoryAdapter`, `XxxGatewayAdapter`
- DTOs: `XxxRequest`, `XxxResponse`, `XxxDTO`, `XxxResult`

### 13.2 Commits
Conventional Commits:
- `feat(bc-nfe): add pure domain and EmitirNfeUseCase foundation`
- `refactor(bc-auth): consume CORP via contract port`
- `docs: add ARCHITECTURE.md`

---

## 14. Glossário rápido

- **BC / Bounded Context:** fronteira explícita de domínio e linguagem.
- **Aggregate:** raiz que controla consistência interna (ex.: Nfe).
- **VO / Value Object:** tipo imutável com validação (Dinheiro, DocumentoFiscal).
- **Ports & Adapters:** interfaces (ports) + implementações (adapters) para isolar infra.

---
