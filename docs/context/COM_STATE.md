# COM_STATE — Sagnus ERP (Context Handoff) v1

> Objetivo: este documento serve como “pacote de contexto” para iniciar um novo chat/módulo sem perder histórico.
> Preencha o que souber. O que não souber, deixe em branco.

---

## 0) Identificação do baseline (obrigatório)
- **Data/Hora do baseline:** 2025-12-19
- **Branch/Tag (se houver):** 
- **Commit hash (se houver):** 
- **ZIP enviado:**  sagnus_20251219.zip
- **Módulos incluídos no ZIP:** sagnus-bc-gateway, sagnus-bc-auth, sagnus-bc-corp, sagnus-bc-corp-contracts,sagnus-bc-nfe, 
                                sagnus-platform-security, sagnus-paltform-web, sagnus-shared-api-error, sagnus-shared-kernel

---

## 1) Visão geral do produto
- **Nome do sistema:** Sagnus ERP
- **Objetivo:** Migração/modernização do legado (Oracle Forms/PLSQL) para arquitetura modular/DDD.
- **Stack atual:** Java 21, Spring Boot, Maven multi-módulo, PostgreSQL (alternativo Oracle), RabbitMQ (fila de trabalho), Flyway.
- **Padrões importantes já decididos:**
  - Nomenclatura Oracle (prefixos por módulo, PT-BR, 31 chars, constraints PK/FK/UK/IX etc.)
  - Lombok como padrão
  - Eventos + Outbox transacional
  - XML NFe: layout NFE40 e RTC2025 por feature-flag

---

## 2) Módulos (Bounded Contexts) e status
> Liste os BCs e o “nível de maturidade”.

| BC / Módulo | Artefato Maven | Status | Observações |
|---|---|---|---|
| NFe | sagnus-bc-nfe | ✅ Estável (pipeline assíncrono) | Outbox+Rabbit+Retry/DLQ+Idempotência, XML NFE40/RTC |
| CORP (cadastro/empresa/filiais) | sagnus-bc-corp (ou nome real) | ⬜ Em construção | Precisa completar tabelas e DDD |
| ... | ... | ... | ... |

---

## 3) Banco de dados — “como está hoje”
### 3.1) SGBD e conexão
- **SGBD:** PostgreSQL 18.1 on x86_64-windows, compiled by msvc-19.44.35219, 64-bit
- **Schema padrão:** sagnus
- **Ferramentas:** pgAdmin / psql / Flyway

### 3.2) Fonte da verdade do schema
Escolha uma (marque X):
- [ ] Flyway (migrations em `db/migration`)
- [X] DDL exportado do banco (pg_dump/pgAdmin)
- [ ] Modelagem externa (ERD/Docs)

### 3.3) Como enviar as tabelas para o próximo chat (recomendado)
Inclua no ZIP:
- `docs/db/schema_dump.sql` (export do schema real)
- ou `docs/db/ddl/*.sql` por módulo
- `docs/db/migrations/` (Flyway) se for a fonte da verdade

Comando recomendado (Postgres):
```bash
pg_dump -s -n sagnus -f docs/db/schema_dump.sql <sua_connection_string>
```

---

## 4) CORP — escopo e regras (para o próximo chat)
> Descreva o que é “CORP” e quais tabelas fazem parte.

### 4.1) Escopo funcional (CORP)
- Cadastro de empresa (matriz/filiais)
- Endereços
- Contatos
- Parametrizações gerais
- (outros…)

### 4.2) Tabelas esperadas (liste)
> Coloque o nome EXATO das tabelas/colunas como estão no banco.

- `corp_empresa` (ou real): ...
- `corp_filial`: ...
- `corp_endereco`: ...
- `corp_contato`: ...
- `corp_parametro`: ...

### 4.3) Chaves naturais / regras de unicidade
- Empresa: CNPJ único?
- Filial: CNPJ+IE?
- Endereço: (empresa_id, tipo) único?
- Contato: (empresa_id, email) único?

---

## 5) Integrações e eventos
### 5.1) Eventos principais já existentes
- `NfeEmitidaEvent`
- `NfeXmlGeradoEvent`
- `NfeXmlAssinadoEvent`
- `NfeAutorizadaEvent`
- `RtcIbsCbsReconciledEvent`

### 5.2) Outbox/Rabbit (resumo)
- Outbox transacional: ✅
- Worker Rabbit: ✅
- Retry TTL + DLQ + redrive admin: ✅
- Idempotência (Inbox): ✅
- Observabilidade (Actuator + Prometheus): ✅
- Hardening (token admin + retenção/limpeza): ✅

---

## 6) Como rodar (dev)
- `mvn test`
- `docker rabbitmq` (se usar)
- `application-local.yml` (onde ficam flags)
- Principais flags:
  - `sagnus.nfe.xml.layout = NFE40 | RTC2025`
  - `sagnus.nfe.outbox.transport = RABBIT`
  - `sagnus.nfe.worker.enabled = true`
  - `sagnus.nfe.sefaz = FAKE`
  - `sagnus.nfe.signer = FAKE`

---

## 7) “O que você quer que eu faça” no próximo chat (o pedido)
> Seja bem objetivo e em bullets.

- [ ] Ler o ZIP atualizado e o schema dump
- [ ] Mapear tabelas do CORP para Aggregates/Entities/Value Objects (DDD)
- [ ] Criar módulo Maven `sagnus-bc-corp` (se ainda não existir)
- [ ] Gerar JPA entities + migrations Flyway alinhadas ao banco real
- [ ] Criar ports/adapters para repositórios
- [ ] Criar endpoints iniciais (CRUD) + validações
- [ ] Documentar `CORP_STATE.md`

---

## 8) Anexos recomendados no ZIP (checklist)
- [ ] `COM_STATE.md` (este arquivo preenchido)
- [ ] `docs/db/schema_dump.sql` (ou export)
- [ ] `docs/context/` (estados por módulo: `NFE_STATE.md`, `CORP_STATE.md`, etc.)
- [ ] `application-local.yml` / `.env.example` (sem senhas)
- [ ] `README.md` com “como rodar”

---

## 9) Notas e decisões recentes (para evitar regressões)
> Liste mudanças que você já fez e não quer que eu “desfaça” sem querer.

- Ex.: Classe `DocumentoFiscal` é a única usada (não existe `Documento`)
- Ex.: `Nfe` tem construtor apenas com (emitente, destinatario); `id` nasce depois
- Ex.: Adapters XML devem ser null-safe para `id`
- Ex.: Jackson deve registrar `JavaTimeModule` (Instant)
- Ex.: Worker Rabbit com retry TTL + DLQ + headers de erro

---

## 10) Mini “glossário interno” (opcional)
- TTL: Time To Live (expiração)
- ACK/NACK: confirmação positiva/negativa
- DLQ: dead-letter queue
- Outbox: tabela transacional de eventos
- Inbox: idempotência de consumo
