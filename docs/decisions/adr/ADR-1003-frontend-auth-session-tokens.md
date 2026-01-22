# ADR-1003 — Frontend Auth, Sessão e Tokens

- **Status:** Accepted
- **Data:** 2026-01-22
- **Contexto:** Sagnus ERP – Frontend (Shell + ADM)

## Contexto
O Sagnus ERP é um sistema de negócio com dados sensíveis. O frontend precisa garantir:
- Segurança contra roubo de sessão (XSS)
- Boa experiência do usuário (renovação silenciosa)
- Alinhamento com o API Gateway e BC ADM

## Decisão
- Access Token (JWT) de curta duração (10–20 min)
- Refresh Token de longa duração
- Refresh Token armazenado em **cookie HttpOnly**
- Access Token mantido **apenas em memória**
- Renovação automática via `/adm/auth/refresh`
- Endpoint `/adm/me` para identidade e permissões

## Fluxo
1. Login → gera access token + set-cookie refresh
2. Requests usam access token
3. 401 → tenta refresh → repete request
4. Falha no refresh → logout

## Consequências
- Maior segurança
- Necessidade de CORS + credentials no gateway
- Implementação de fila de refresh no frontend
