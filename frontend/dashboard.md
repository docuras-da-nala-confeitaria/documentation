---
layout: default
title: Dashboard
parent: Frontend
nav_order: 2
---

# Dashboard

- 4 chamadas em paralelo via `Promise.allSettled` (resumo, evolução,
  vendas por sabor, vendas por canal) — **falha de uma não trava as
  demais**, cada bloco tem seu próprio "Tentar novamente".
- Skeletons enquanto carrega.
- Card "Itens com estoque baixo" só é clicável quando > 0; abre
  `ModalEstoqueBaixo` (busca `GET /estoque/alertas` sob demanda,
  acessível: foco automático, fecha com Esc/clique fora).
- `CardMeta`: barra de progresso clampada entre 0–100% mesmo que o backend
  retorne valor fora da faixa.
