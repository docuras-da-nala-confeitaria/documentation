---
layout: default
title: Clientes
parent: Frontend
nav_order: 3
---

# Clientes

- Busca por nome/telefone dispara `GET /clientes?busca=` a cada tecla
  (**sem debounce**).
- **Não existe `GET /clientes/{id}` no backend** — a tela de edição busca
  a lista inteira e filtra pelo id no client.
- `FormCliente`: nome e telefone obrigatórios (trim); endereço vazio vira
  `null`; mensagem de sucesso some após 3s.
