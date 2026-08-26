---
layout: default
title: Recursos de Produção
parent: Backend
nav_order: 6
---

# Recursos de Produção (`/recursos-producao`)

Equipamentos/mão de obra (`custoPorHora`), usados em receitas via
`tempoMinutos`.

- `GET /recursos-producao` / `GET /recursos-producao/{id}` —
  listagem/busca.
- `POST /recursos-producao` — cria (`nome`, `custoPorHora`).
- `PATCH /recursos-producao/{id}` — **se `custoPorHora` mudar**, dispara
  `CustoRecursoAtualizadoEvent` → recálculo automático de precificação de
  todos os produtos cuja receita usa esse recurso.
