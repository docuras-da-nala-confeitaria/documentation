---
layout: default
title: Dashboard
parent: Backend
nav_order: 12
---

# Dashboard (`/dashboard`) — somente leitura

- `GET /dashboard/resumo` — faturamento/despesas/resultado do mês,
  quantidade de pedidos, unidades vendidas (pedidos **entregues**), produto
  mais vendido do mês, dia de maior faturamento, itens com estoque
  baixo/comprar-em-breve, meta do mês e % atingido (`null` se sem meta ou
  meta=0).
- `GET /dashboard/evolucao` — série de faturamento dos últimos **6 meses**,
  só pedidos `ENTREGUE`.
- `GET /dashboard/vendas-por-sabor` — unidades vendidas por produto
  (entregues), ordenado desc.
- `GET /dashboard/vendas-por-canal` — total vendido (R$) por canal
  (entregues), ordenado desc.
