# ADR-1003 — Frontend Auth, Sessão e Tokens

- **Status:** Accepted
- **Data:** 2026-01-22
- **Escopo:** Frontend (Shell + BC ADM)

## Contexto
O Sagnus ERP é um sistema corporativo com dados sensíveis. O frontend precisa garantir:
- Segurança contra roubo de sessão (XSS)
- Renovação transparente de sessão
- Integração limpa com API Gateway e BC ADM

## Decisão
- Access Token (JWT) de curta duração (10–20 minutos)
- Refresh Token de longa duração
- Refresh Token armazenado em **cookie HttpOnly**
- Access Token mantido **apenas em memória**
- Renovação automática via endpoint `/adm/auth/refresh`
- Endpoint `/adm/me` para identidade, roles e permissões

## Consequências
- Segurança elevada contra XSS
- Necessidade de CORS com credentials no gateway
