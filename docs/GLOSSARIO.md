# Glossário Arquitetural – Sagnus ERP

Este glossário define os principais termos arquiteturais utilizados no projeto Sagnus ERP.
Ele complementa os ADRs e serve como referência comum para desenvolvedores, arquitetos e agentes de IA.

---

## ADR (Architectural Decision Record)
Documento que registra uma decisão arquitetural relevante do sistema.

Um ADR descreve:
- O contexto da decisão
- A decisão tomada
- Alternativas consideradas
- Consequências (positivas e negativas)

Exemplo: ADR-0001 – Baseline Arquitetural (DDD + Hexagonal).

---

## Bounded Context (BC)
Delimitação clara de um contexto de negócio dentro do sistema, com:
- Modelo de domínio próprio
- Linguagem ubíqua específica
- Independência de outros BCs

No Sagnus, exemplos de BCs:
- bc-corp
- bc-adm
- bc-estoque
- bc-nfe

---

## Domain
Camada central do sistema onde reside a lógica de negócio pura.

Características:
- Java puro
- Sem dependência de frameworks
- Contém entidades, value objects, domain services e regras de negócio

---

## Application
Camada responsável por orquestrar casos de uso.

Características:
- Implementa Use Cases
- Define Ports (interfaces)
- Não contém regras de negócio complexas
- Não depende de infraestrutura

---

## Infrastructure
Camada responsável por detalhes técnicos e frameworks.

Exemplos:
- Persistência (JPA, JDBC)
- Mensageria
- Integrações externas
- Configuração de Spring

Implementa os adapters definidos pelos ports.

---

## API
Camada de entrada do sistema.

Responsável por:
- Expor endpoints REST ou GraphQL
- Validar requests
- Mapear dados para Use Cases

Não deve conter regras de negócio nem acessar infraestrutura diretamente.

---

## Port
Interface que define um contrato entre a Application/Domain e o mundo externo.

Tipos comuns:
- Ports de saída (repositórios, gateways)
- Ports de entrada (casos de uso)

---

## Adapter
Implementação concreta de um Port.

Normalmente localizada na Infrastructure.
Exemplo: Adapter JPA que implementa um Repository Port.

---

## Contract
Objeto ou módulo que define dados compartilhados entre Bounded Contexts.

Usado para:
- Eventos
- DTOs públicos
- Integrações entre BCs

Evita dependência direta entre domínios.

---

## BFF (Backend for Frontend)
Camada intermediária entre frontend e backend.

No Sagnus:
- Implementado no API Gateway
- Orquestra chamadas entre BCs
- Não contém regras de negócio

---

## Hexagonal Architecture
Estilo arquitetural que separa:
- Núcleo (Domain + Application)
- Interfaces externas (API, Infrastructure)

Baseado em Ports e Adapters.

---

## Shared Kernel
Conjunto pequeno de conceitos compartilhados entre BCs.

Deve ser:
- Estável
- Muito limitado
- Usado com cautela

---

## ArchUnit
Ferramenta de testes usada para validar regras arquiteturais no código.

Permite garantir automaticamente que:
- Domain não depende de frameworks
- Application não depende de Infrastructure
- BCs não se acoplam indevidamente

---

## CI Gate
Regra no pipeline de integração contínua que bloqueia o merge de código caso testes ou regras arquiteturais falhem.

Garante a aplicação prática dos ADRs.

---
