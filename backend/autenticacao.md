---
layout: default
title: Autenticação
parent: Backend
nav_order: 1
---

# Autenticação (`/auth`)

| Rota | Método | Descrição |
|---|---|---|
| `/auth/login` | POST | Login com e-mail/senha |
| `/auth/logout` | POST | No-op (JWT stateless) |
| `/auth/me` | GET | Retorna usuário autenticado a partir do token |
| `/auth/esqueci-senha` | POST | Solicita redefinição de senha |
| `/auth/redefinir-senha` | POST | Efetiva a redefinição com token |

**POST /auth/login** — busca usuário por e-mail (401 genérico "E-mail ou
senha inválidos." se não existir ou senha não bater); 401 "Usuário
inativo." se `ativo=false`; compara hash BCrypt; gera JWT com claims
`email`/`papel`, validade configurável (`app.jwt.expiracao-horas`).

**POST /auth/logout** — sem efeito no servidor (stateless), retorna 204.

**GET /auth/me** — exige header `Authorization: Bearer <token>`; 401 se
ausente/mal formatado; valida assinatura/expiração; 404 se usuário não
existir mais.

**POST /auth/esqueci-senha** — **sempre retorna 204**, exista ou não o
e-mail (proteção contra enumeração de usuários). Se existir: invalida
tokens de reset anteriores não usados, gera token UUID com validade de
**1 hora**, publica notificação via SNS com link de redefinição.
Importante: o SNS **não envia e-mail ao endereço dinâmico do usuário** —
apenas notifica assinantes fixos do tópico (não é entrega real de e-mail;
falha de publicação é apenas logada).

**POST /auth/redefinir-senha** — valida token (existência, não usado, não
expirado — mensagens específicas para cada caso), atualiza `senhaHash`
(BCrypt) e marca token como usado.
