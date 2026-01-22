# Sagnus ERP - Technical Documentation

**Version:** 1.0
**Date:** 2025-12-19

---

## 1. Introduction

**Sagnus ERP** is a next-generation, modular Enterprise Resource Planning system designed to support complex fiscal domains (NF-e, IBS/CBS) and allow for independent evolution of business modules. It is built upon the principles of **Domain-Driven Design (DDD)** and **Clean Architecture**.

### Core Objectives

- **Modularity**: Independent Bounded Contexts (BCs) that can evolve separately.
- **Decoupling**: Strict boundaries between domains to prevent "spaghetti code".
- **Scalability**: Prepared for microservices architecture in the future.
- **Maintainability**: Pure domain logic isolated from frameworks.

---

## 2. Architecture Overview

The system follows a **Hexagonal Architecture (Ports & Adapters)** approach within each Bounded Context.

### 2.1 Architectural Layers

1.  **Domain**: The core business logic. Contains Entities, Value Objects, and Aggregates. **No frameworks allowed** (pure Java).
2.  **Application**: Orchestrates use cases. Defines Ports (interfaces) for external dependencies.
3.  **Infrastructure**: Implements Ports (Adapters). Contains database repositories (JPA), external API clients, and messaging adapters.
4.  **API**: The entry point for the application (REST Controllers). Handles HTTP requests/responses and DTO conversion.

### 2.2 Bounded Contexts (BCs)

The system is divided into autonomous business units:

- **CORP (`sagnus-bc-corp`)**: Corporate data source of truth (Person, Product).
- **AUTH (`sagnus-bc-auth`)**: Authentication and Authorization (Identity Provider).
- **NFE (`sagnus-bc-nfe`)**: Electronic Invoicing (Fiscal Domain).

### 2.3 Communication Between BCs

- **Synchronous**: Via **Contracts** (`*-api` modules).
  - Example: `bc-auth` depends on `bc-corp-contracts` to query user details. It does **NOT** depend on `bc-corp` implementation.
- **Asynchronous**: Via Domain Events (future implementation).

---

## 3. Module Structure

The project uses a multi-module Maven structure.

### 3.1 Bounded Context Modules

Each BC typically follows this internal package structure:

```
com.slifesys.sagnus.<context>
├── domain          # Entities, VOs, Domain Services, Exceptions
├── application     # UseCases, Ports (Input/Output)
├── infrastructure  # JPA Repositories, External Clients, Mappers
└── api             # REST Controllers, DTOs
```

### 3.2 Contract Modules (`*-api`)

These modules define the public API of a BC for other Java modules to consume.

- **Contains**: Interfaces (Ports), DTOs, Enums.
- **Goal**: Allow compile-time safety without implementation coupling.

### 3.3 Platform & Shared Modules

- **`sagnus-platform-security`**: Centralized security logic.
  - Stateless JWT authentication.
  - Common security filters and configuration.
- **`sagnus-shared-api-error`**: Standardized error handling.
  - Defines `ErrorResponse` structure.
  - Global Exception Handler base.
- **`sagnus-platform-web`**: Common web infrastructure.

---

## 4. Technical Stack

- **Language**: Java 21
- **Build System**: Maven 3.9+
- **Framework**: Spring Boot 3.3.4
  - Used strictly in `Infrastructure` and `API` layers.
- **Database**: PostgreSQL
- **Persistence**: JPA / Hibernate (encapsulated in Infrastructure).
- **Security**: Spring Security + JWT (jjwt library).

---

## 5. Development Guidelines

### 5.1 Rules of Thumb

1.  **Domain First**: Always start modeling the Domain (Entities/VOs) before thinking about tables or JSON.
2.  **Dependency Rule**: Inner layers (Domain) must NOT depend on outer layers (Infrastructure/API).
3.  **No Cross-Database Access**: A BC must never access another BC's tables directly.

### 5.2 Naming Conventions

- **Use Cases**: `VerbSubjectUseCase` (e.g., `EmitirNfeUseCase`).
- **Ports**: `SubjectActionPort` (e.g., `NfeRepositoryPort`, `CorpIntegrationPort`).
- **Implementations**: `SubjectActionAdapter` or `JpaSubjectRepository`.

### 5.3 Error Handling

All APIs must return errors in the standard format defined in `sagnus-shared-api-error`:

```json
{
  "timestamp": "2025-12-19T10:00:00Z",
  "status": 400,
  "errorType": "VALIDATION_ERROR",
  "code": "NFE-001",
  "message": "Invalid tax calculation",
  "correlationId": "..."
}
```

---

## 6. Security Model

- **Stateless**: No server-side sessions.
- **JWT**: Tokens carry identity and claims.
- **Flow**:
  1.  Client sends credentials to `bc-auth`.
  2.  `bc-auth` validates and issues Access + Refresh Tokens.
  3.  Client sends Access Token in `Authorization: Bearer` header for subsequent requests.
  4.  `sagnus-platform-security` intercepts requests to validate the token signature and extract user context.
