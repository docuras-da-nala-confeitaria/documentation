---
layout: default
title: Fluxo de Caixa
parent: Frontend
nav_order: 9
---

# Fluxo de Caixa

- Seletor de mês independente dos filtros da lista de lançamentos.
- Ambas chamadas usam `sequenciaRef` contra race condition.
- **Regra central: origem do lançamento determina as ações disponíveis**:
  - `manual`: editável e excluível (com confirmação).
  - `pedido`: não editável, link "Ver pedido".
  - `entrada_estoque`: não editável, link "Ver ingrediente".
- `ModalLancamentoCaixa`: categoria obrigatória, valor > 0, data
  obrigatória.
