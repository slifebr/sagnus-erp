# NFE_STATUS — Sagnus BC NF-e (estado atual)

Data: 2025-12-21  
Contexto: BC NF-e com pipeline assíncrono via Outbox + Rabbit Worker, geração de XML e preparação p/ SEFAZ (fake por enquanto).

---

## 1) Objetivo do módulo (escopo atual)

- Emitir NF-e (MVP) com:
  - Domínio (Nfe, NfeItem, Emitente, Destinatario, TributosItem)
  - Cálculo e normalização RTC de IBS/CBS
  - Geração XML (NFE40 e RTC2025)
  - Pipeline assíncrono via Outbox + Rabbit Worker
  - Preparação para assinatura e envio SEFAZ (ainda FAKE)

---

## 2) Decisões arquiteturais consolidadas

### 2.1 Outbox (persistência e despacho)

- Evento de domínio `NfeEmitidaEvent` é persistido em tabela de outbox.
- Dispatcher/outbox publica envelope no transporte configurado:
  - `transport=RABBIT` (ativo)
  - Outros transportes podem existir via `OutboxTransportPort`.

### 2.2 Rabbit Worker (consumo e execução)

- Worker consome fila principal, aplica idempotência (Inbox), executa handler por `eventType`.
- Handlers pluggable: `eventType -> handler` (separados).
- Retentativa (retry) e redrive (DLQ/parking) implementados via estratégia:
  - contagem de deliveries
  - TTL retry
  - fallback para fila de erro após exceder limite

### 2.3 Assíncrono de verdade

- “Gerar XML” foi movido para o worker (fora do request/transaction path).
- Não há listener “in-process” fazendo trabalho pesado além do outbox writer.

---

## 3) IBS/CBS (RTC) — estado atual

### 3.1 Modelo

- `TributosItem` suporta IBS e CBS via Optional/value objects.
- Normalização RTC implementada (reconciliar base/aliquota/valor) com tolerância configurável.
- Log de normalização em WARN (quando ajusta) com dados base/aliq/valor/expected/diff/tol.

### 3.2 Emissão no XML

- **NFE40**: não quebra XSD NF-e 4.00
  - IBS/CBS transportado em `<infAdProd>` com prefixo `RTC:IBS[...] CBS[...]` (schema-safe).
- **RTC2025**: emite grupos RTC dentro de `<imposto>` (quando layout RTC ativo).

---

## 4) XML / Layouts

### 4.1 Adapters

- `NfeXmlGeneratorNfe40Adapter` (default)
- `NfeXmlGeneratorRtc2025Adapter` (quando habilitado via property)

### 4.2 Flags principais

- `sagnus.nfe.xml.layout = NFE40 | RTC2025`
- (NFE40 é o padrão / matchIfMissing)

---

## 5) SEFAZ — estado atual (dev)

### 5.1 UF e ambiente fixos (DEV)

- Fixado para **SP / Homologação**
  - `cUF = 35`
  - `tpAmb = 2`
- URLs de WS apontadas para SP/Homolog (ainda sem chamada real, pois SEFAZ=FAKE).

### 5.2 Certificado

- Implementado carregamento de **PFX por diretório**:
  - objetivo: você coloca um A1 real depois, sem mudar código
  - dev pode rodar com PFX “mock”/placeholder (sem envio real ainda)

### 5.3 Envio

- **FAKE** por enquanto:
  - estrutura já preparada para ligar assinatura + transmissão quando entrar certificado real + endpoints reais.

---

## 6) Numeração / IDE / Chave NF-e

- Evoluções já plugadas:
  - Numeração e montagem de IDE (nNF/serie etc. conforme MVP)
  - Geração/armazenamento de chave no domínio e no XML/IDE
  - (assinatura digital ainda não ativa)

---

## 7) Observabilidade e Hardening (nível “pré-produção”)

### 7.1 Logs e correlação

- CorrelationId propagado no envelope do outbox/worker.
- Log estruturado no publish e no consumo.
- Auditoria de eventos (quando habilitada por config).

### 7.2 Resiliência

- Retry/Backoff/Redrive
- Idempotência via Inbox (não reprocessar eventId já aplicado)
- Dead-letter / parking para mensagens problemáticas

---

## 8) Banco (tabelas principais)

### 8.1 Outbox

- Tabela de outbox com:
  - eventId (String)
  - eventType
  - occurredAt
  - correlationId
  - payloadJson
  - status / attempts / nextAttemptAt etc. (conforme versão do patch)

### 8.2 Inbox (Idempotência)

- `nfe_inbox_processed`
  - `event_id` **String (varchar)**
  - `processed_at` timestamp

> Importante: eventId é String no fluxo inteiro.  
> Se alguma base tinha BIGINT em DEV, ajustar o tipo para varchar.

### 8.3 Audit

- `nfe_audit_event` (quando auditoria ligada)

---

## 9) Configuração (properties) — pontos sensíveis

### 9.1 Rabbit Outbox/Worker

- exchange / routingKey / queue principal
- retryQueue / retryTtlMs / maxDeliveries / prefetch (dependendo da versão aplicada)
- redriveRoutingKey / dlq (quando aplicável)

### 9.2 Layout XML

- `sagnus.nfe.xml.layout`

### 9.3 Auditoria

- `sagnus.nfe.audit.enabled` (ou equivalente da classe de properties)

---

## 10) Problemas já resolvidos durante a evolução

- Jackson não serializava `Instant` no envelope:
  - corrigido com `jackson-datatype-jsr310` / configuração do ObjectMapper.
- Erros “cannot find symbol log”:
  - corrigido com Lombok `@Slf4j` / imports e/ou dependência lombok correta.
- Divergências de assinatura de port/adapters (ex.: corp adapter):
  - alinhado contrato (override ok).
- `eventId` String vs Long no Inbox:
  - padronizado para String.

---

## 11) Como rodar / testar (fluxo esperado)

1. API recebe comando “Emitir NFe”
2. Use case salva NFe + gera evento
3. Listener/outbox writer persiste evento na Outbox
4. Dispatcher publica no Rabbit
5. Worker consome:
   - valida idempotência (Inbox)
   - escolhe handler por eventType
   - gera XML e executa pipeline
   - grava Inbox como processado
6. (Futuro) assina + transmite SEFAZ real

---

## 12) Próximas etapas sugeridas (para fechar “produção real”)

Prioridade alta:

1. Assinatura real (XMLDSig) com A1 PFX (já temos loader por diretório)
2. Integração real com WS SP/Homolog (SOAP + MTOM/headers conforme necessário)
3. Persistir retorno SEFAZ (protNFe, recibo, cStat, xMotivo)
4. Reprocessamento operacional:
   - painel de fila (retry/dlq)
   - comando “redrive”
5. Observabilidade de produção:
   - métricas (Micrometer/Prometheus)
   - tracing (OpenTelemetry)
6. Validação XSD + regras de negócio (antes do envio)

---

## 13) Nota operacional (importante para novos patches)

Para evitar regressão:

- antes de novos passos grandes, gerar ZIP atualizado do workspace
- manter um arquivo `docs/context/_SNAPSHOT.md` com:
  - hash do commit
  - mudanças locais pendentes
  - “quebras” de contrato (ports/properties/tipos)
