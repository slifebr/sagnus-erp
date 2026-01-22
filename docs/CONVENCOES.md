# Sagnus ERP — Convenções de Arquitetura (DDD + Hexagonal)

**Data:** 2026-01-19  
**Objetivo:** Definir o padrão oficial de organização de pacotes, persistência e nomenclatura para todos os Bounded Contexts do projeto Sagnus.

---

## 1. Estrutura de Pacotes (Canônica)

Este projeto adota **DDD + Arquitetura Hexagonal** para manter o domínio independente de frameworks e banco de dados.

### Princípios

- **Domain** contém regras e contratos (ports). Não depende de Spring/JPA/Jackson.
- **Application** orquestra casos de uso (use cases). Conhece o domínio e suas interfaces.
- **Infrastructure** contém detalhes técnicos (adapters): persistência, mensageria, HTTP clients, cache etc.
- A pasta **infrastructure/persistence** só existe quando há persistência real (JPA/JDBC/etc).
- A pasta **infrastructure/repository** é reservada a stubs/in-memory/fakes (quando não há DB).

### Layout Padrão

```
com.slifesys.sagnus.<bc>/
  api/
    controller/                # REST controllers
    dto/                       # Request/Response DTOs
    mapper/                    # DTO <-> Application (Command/Query)
    graphql/                   # opcional/futuro
  application/
    usecase/                   # Use cases (orquestração)
    service/                   # Application services (se necessário)
    port/
      in/                      # Ports de entrada (opcional)
      out/                     # Ports de integração externa (ver ADR-0010)
  domain/
    model/                     # Aggregates, Entities, Value Objects
    repository/                # PORTS de persistência (interfaces)
    service/                   # Domain services
    event/                     # Domain events
    exception/                 # Domain exceptions
  infrastructure/
    config/                    # Spring config, beans
    event/                     # Outbox listener, handlers, publisher
    persistence/               # SÓ SE houver banco
      entity/                  # JPA Entities
      repository/              # Spring Data (JpaRepository)
      mapper/                  # Entity <-> Domain
      adapter/                 # Implementa domain.repository.*
    repository/                # SOMENTE stubs (InMemory*, Fake*)
    http/                      # Clients externos (Feign/WebClient)
    messaging/                 # Rabbit/Kafka adapters
```

---

## 2. Regras de Persistência

### 2.1. Ports do Domínio

✅ **Localização:** `domain/repository/`

**Exemplo:**
```java
// domain/repository/ProdutoRepository.java
public interface ProdutoRepository {
    Produto findById(ProdutoId id);
    void save(Produto produto);
}
```

📌 **Regra:** Ports de repositório são **contratos do domínio**, não do Spring Data.

### 2.2. Spring Data (JPA)

✅ **Localização:** `infrastructure/persistence/repository/*JpaRepository.java`

**Exemplo:**
```java
// infrastructure/persistence/repository/ProdutoJpaRepository.java
public interface ProdutoJpaRepository extends JpaRepository<ProdutoEntity, Long> {
}
```

📌 **Regra:** Spring Data é um **detalhe de infraestrutura**, não um port do domínio.

### 2.3. Adapter (Implementação do Port)

✅ **Localização:** `infrastructure/persistence/adapter/*RepositoryImpl.java`

**Exemplo:**
```java
// infrastructure/persistence/adapter/ProdutoRepositoryImpl.java
@Component
public class ProdutoRepositoryImpl implements ProdutoRepository {
    private final ProdutoJpaRepository jpaRepository;
    private final ProdutoEntityMapper mapper;
    
    @Override
    public Produto findById(ProdutoId id) {
        return jpaRepository.findById(id.getValue())
            .map(mapper::toDomain)
            .orElse(null);
    }
}
```

📌 **Regra:** O adapter **implementa o port do domínio** e usa Spring Data internamente.

### 2.4. JPA Entity

✅ **Localização:** `infrastructure/persistence/entity/*Entity.java`

**Exemplo:**
```java
// infrastructure/persistence/entity/ProdutoEntity.java
@Entity
@Table(name = "produto")
public class ProdutoEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    // ...
}
```

📌 **Regra:** Entidades JPA **não devem vazar para o domínio**.

### 2.5. Mapper (Entity <-> Domain)

✅ **Localização:** `infrastructure/persistence/mapper/*EntityMapper.java`

**Exemplo:**
```java
// infrastructure/persistence/mapper/ProdutoEntityMapper.java
@Mapper(componentModel = "spring")
public interface ProdutoEntityMapper {
    Produto toDomain(ProdutoEntity entity);
    ProdutoEntity toEntity(Produto domain);
}
```

📌 **Regra:** Mappers de persistência ficam em **infrastructure**, não em domain.

### 2.6. Quando Criar `infrastructure/persistence`

✅ **Criar quando:**
- O BC possui tabelas no banco de dados
- Há persistência real (JPA/JDBC/MyBatis)

❌ **NÃO criar quando:**
- O BC ainda não tem banco de dados
- Está usando apenas stubs/in-memory

📌 **Alternativa:** Use `infrastructure/repository/InMemory*` para stubs.

---

## 3. Convenções de Nomenclatura

| Tipo | Sufixo/Padrão | Localização | Exemplo |
|------|---------------|-------------|---------|
| **Port do domínio** | `Repository` | `domain/repository/` | `ProdutoRepository` |
| **Spring Data** | `JpaRepository` | `infrastructure/persistence/repository/` | `ProdutoJpaRepository` |
| **Adapter** | `RepositoryImpl` | `infrastructure/persistence/adapter/` | `ProdutoRepositoryImpl` |
| **JPA Entity** | `Entity` | `infrastructure/persistence/entity/` | `ProdutoEntity` |
| **Mapper Entity-Domain** | `EntityMapper` | `infrastructure/persistence/mapper/` | `ProdutoEntityMapper` |
| **Mapper DTO-App** | `DtoMapper` ou `ApiMapper` | `api/mapper/` | `ProdutoDtoMapper` |
| **Use Case** | `UseCase` | `application/usecase/` | `CriarProdutoUseCase` |
| **Domain Service** | `Service` | `domain/service/` | `CalculadoraPrecoService` |

---

## 4. Localização de Ports por Tipo (ADR-0010)

Consulte `DECISIONS.md` ADR-0010 para detalhes completos.

| Tipo de Port | Localização | Exemplo |
|--------------|-------------|---------|
| **Repositório** | `domain/repository/` | `ProdutoRepository` |
| **Integração externa** | `application/port/out/` | `SefazPort`, `EmailPort` |
| **Caso de uso** (opcional) | `application/port/in/` | `CriarProdutoPort` |

📌 **Regra:** Ports de **repositório** são sempre do domínio. Ports de **integração externa** podem ficar em `application/port/out/`.

---

## 5. Checklist de Revisão por BC

Use este checklist para validar se um BC está seguindo o padrão:

### DDD / Hexagonal

- [ ] Domain não depende de Spring, JPA, Jackson, Feign, etc.
- [ ] Ports estão em `domain/repository/` (repositórios) ou `application/port/out/` (integrações)
- [ ] `infrastructure/persistence/entity` não é importada em `domain/` ou `application/`

### Persistência

- [ ] `infrastructure/persistence/repository/*JpaRepository` existe apenas se há DB
- [ ] `infrastructure/persistence/adapter/*Impl` implementa exatamente o port do domínio
- [ ] Mapper Entity <-> Domain existe (evitar "domain com annotation JPA")

### Stubs

- [ ] Se o BC ainda não tem banco: `infrastructure/repository/InMemory*` pode existir
- [ ] Se o BC ainda não tem banco: **não existe** `infrastructure/persistence/*`

### API

- [ ] Controller usa `application/usecase`
- [ ] DTOs não vazam para Domain
- [ ] Mappers DTO <-> Application estão em `api/mapper/`

---

## 6. Exemplos Práticos

### 6.1. BC com Persistência (bc-estoque)

✅ **Estrutura atual (após ajuste):**

```
com.slifesys.sagnus.estoque/
  domain/
    repository/EstoqueLocalRepository.java          # Port do domínio
  infrastructure/
    persistence/
      entity/EstoqueLocalEntity.java                # JPA Entity
      repository/EstoqueLocalJpaRepository.java     # Spring Data
      mapper/EstoqueLocalEntityMapper.java          # Entity <-> Domain
      adapter/EstoqueLocalRepositoryImpl.java       # Implementa port
```

📌 **Mudança aplicada:** Mover `RepositoryImpl` de `repository/` para `adapter/` para separar claramente:
- `repository/` = Spring Data
- `adapter/` = Implementação do port do domínio

### 6.2. BC sem Persistência (bc-fina-base)

✅ **Estrutura atual:**

```
com.slifesys.sagnus.fina.base/
  domain/
    repository/ContaRepository.java                 # Port do domínio
  infrastructure/
    repository/InMemoryContaRepository.java         # Stub in-memory
```

📌 **Regra:** Não criar `infrastructure/persistence/` até que haja tabelas reais.

📌 **Quando entrar DB:** Criar `persistence/{entity,repository,mapper,adapter}` seguindo o padrão do bc-estoque.

---

## 7. Geração de Novos BCs (new-bc-from-sql)

### Se o SQL tiver tabelas do BC

Gerar automaticamente:

```
domain/repository/<Entidade>Repository.java
infrastructure/persistence/entity/<Entidade>Entity.java
infrastructure/persistence/repository/<Entidade>JpaRepository.java
infrastructure/persistence/mapper/<Entidade>EntityMapper.java
infrastructure/persistence/adapter/<Entidade>RepositoryImpl.java
```

### Se o BC for "base / sem SQL"

Gerar:

```
infrastructure/repository/InMemoryExampleRepository.java  # opcional
```

**NÃO gerar** `infrastructure/persistence/`

---

## 8. Referências

- **Decisões arquiteturais:** Consulte `DECISIONS.md` para ADRs completas
- **Regras do Cursor AI:** Consulte `.cursorrules` para diretrizes de desenvolvimento
- **Estrutura de pacotes:** Esta seção (CONVENCOES.md § 1)
- **Persistência:** Esta seção (CONVENCOES.md § 2)

---

## 9. Histórico de Mudanças

- **2026-01-19:** Consolidação do documento, remoção de duplicações, adição de referências a ADRs
- **2025-12-16:** Versão inicial com múltiplas seções (consolidadas nesta versão)
