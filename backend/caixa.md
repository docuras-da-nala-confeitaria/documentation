---
layout: default
title: Caixa
parent: Backend
nav_order: 11
---

# Caixa / Fluxo Financeiro (`/caixa`)

Enums: `TipoLancamento {ENTRADA, SAIDA}`; `OrigemLancamento {MANUAL,
PEDIDO, ENTRADA_ESTOQUE, ...}`.

- `GET /caixa?tipo=&categoria=&dataInicio=&dataFim=` — filtros combináveis
  (categoria case-insensitive).
- `POST /caixa` — `tipo`, `categoria`, `valor`, `data`, `descricao`; sempre
  criado com `origem = MANUAL`.
- `PATCH /caixa/{id}` — **só lançamentos `origem == MANUAL` podem ser
  editados**; erro se tentar editar lançamento automático
  (`PEDIDO`/`ENTRADA_ESTOQUE`).
- `DELETE /caixa/{id}` — mesma regra de origem MANUAL.
- `GET /caixa/resumo?mes=yyyy-MM` — `faturamento` (entradas do mês),
  `despesas` (saídas do mês), `resultado = faturamento - despesas`.
