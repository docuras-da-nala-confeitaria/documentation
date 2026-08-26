---
layout: default
title: Usuários
parent: Backend
nav_order: 2
---

# Usuários (`/usuarios`)

- `GET /usuarios` — lista todos.
- `POST /usuarios` — cria (`nome`, `email`, `senha`, `papel`); e-mail único;
  senha obrigatória e criptografada; usuário sempre nasce `ativo=true`.
- `PATCH /usuarios/{id}` — sem `@Valid`; valida unicidade de e-mail
  (ignorando o próprio); senha só muda se enviada e não vazia.
- `PATCH /usuarios/{id}/status` — ativa/inativa (usado para bloquear login).
