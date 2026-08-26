---
layout: default
title: Produtos
parent: Backend
nav_order: 4
---

# Produtos (`/produtos`)

- `GET /produtos?categoria_id=&status=&modo_producao=` — filtros combináveis.
- `GET /produtos/{id}` — inclui campos calculados: `custoAtual` (via receita,
  `null` se não houver), `lucroPorUnidade = precoVenda - custoAtual`,
  `margemLucroPercentual` (só se `precoVenda != 0`).
- `POST /produtos` — `nome`, `categoriaId` (deve existir), `peso`,
  `precoVenda`, `status`, `modoProducao`.
- `PATCH /produtos/{id}` — sem `@Valid`; substitui todos os campos.
- `PATCH /produtos/{id}/status` — ativa/inativa (não afeta pedidos/estoque
  existentes).

## Canais de Venda (`/canais-venda`)

- `GET /canais-venda?status=` — filtro opcional `ATIVO`/`INATIVO`.
- `GET /canais-venda/{id}` — 404 se não achar.
- `POST /canais-venda` — `nome`, `taxaCanal`, `taxaPagamento`, `status`.
- `PATCH /canais-venda/{id}` — **se `taxaCanal` ou `taxaPagamento`
  mudarem**, dispara evento `CustoCanalVendaAtualizadoEvent` → recálculo
  automático de precificação de todos os produtos cujo **último** cálculo
  usou esse canal. Mudar apenas nome/status não recalcula nada.
