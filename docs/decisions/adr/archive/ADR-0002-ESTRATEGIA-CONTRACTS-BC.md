# ADR-0002 – Estratégia de Contracts entre Bounded Contexts

## Status
ACCEPTED

## Contexto
O Sagnus ERP é estruturado em múltiplos Bounded Contexts (BCs), cada um responsável por um domínio de negócio específico.
Este ADR define a estratégia oficial para comunicação e compartilhamento de dados entre BCs.

## Decisão
A comunicação entre Bounded Contexts será realizada exclusivamente por meio de Contracts explícitos.

## Regras
- Proibida dependência direta entre BCs
- Uso obrigatório de módulos *-contracts
- Contracts não contêm lógica de negócio

## Consequências
BCs independentes, arquitetura governada e evolução segura.
