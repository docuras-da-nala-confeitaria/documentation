---
layout: default
title: Pedidos
parent: Backend
nav_order: 9
---

# Pedidos (`/pedidos`) — módulo mais crítico

Enums: `StatusPedido {EM_PRODUCAO, A_CAMINHO, ENTREGUE, CANCELADO}`;
`StatusPagamento {PENDENTE, PAGO}`.

- `GET /pedidos?canal_venda_id=&status_pedido=&status_pagamento=&data_inicio=&data_fim=`
  — todos filtros opcionais.
- `GET /pedidos/{id}` — 404 se não achar.
- `POST /pedidos` — `clienteId` (deve existir), `canalVendaId` (deve
  existir e estar `ATIVO`), `formaPagamento`, `taxaEntrega` (default 0),
  `desconto` (default 0), `horarioEntrega`, `observacoes`, `itens[]`.
  Número do pedido é sequencial automático (`maior número + 1`, iniciando
  em 1). `valorTotal` sempre **calculado dinamicamente** (não persistido):
  soma itens + taxa de entrega − desconto.
- `PATCH /pedidos/{id}` — taxa de entrega, desconto, horário, observações;
  exige pedido editável.
- `PATCH /pedidos/{id}/status-pagamento` — exige pedido editável.

## `PATCH /pedidos/{id}/status-pedido` — regra mais complexa do sistema

- Exige pedido editável.
- Para mudar para `ENTREGUE`: exige `statusPagamento == PAGO` primeiro
  (senão erro "Só é possível marcar como entregue com o pagamento
  confirmado").
- `processarEntrega`, dentro de uma única transação:
  1. Para cada item: se `SOB_ENCOMENDA`, calcula consumo de ingredientes
     da receita (proporcional à quantidade, agregado entre itens); senão,
     consome do `ProdutoEstoque` (agregado por produto).
  2. Embalagem é baixada **sempre**, independente do modo de produção:
     `POR_UNIDADE` escala pela quantidade; `POR_VENDA` conta **uma vez
     por pedido** (usa `max`, não soma, entre produtos do mesmo pedido);
     se o mesmo ingrediente aparecer em ambas categorias, os totais somam.
  3. Valida estoque suficiente (ingredientes de receita, embalagem,
     produtos prontos) **antes** de qualquer baixa; erro lista tudo que
     falta.
  4. Dá baixa nos ingredientes de receita e embalagem, recalcula status
     de cada um.
  5. Dá baixa no `ProdutoEstoque`, registra movimentação `VENDA`
     vinculada ao pedido.
  6. **Cria lançamento automático no Caixa**: `ENTRADA`, categoria
     "Vendas", valor = total do pedido, origem `PEDIDO`, descrição
     "Pedido #N".
- Para outros status (`EM_PRODUCAO`, `A_CAMINHO`, `CANCELADO`): apenas
  troca o status, sem efeitos colaterais.

- `POST /pedidos/{id}/itens` — adiciona item; exige pedido editável.
- `DELETE /pedidos/{id}/itens/{itemId}` — remove item; exige pedido
  editável; 404 se item não pertence ao pedido.

**Regra transversal**: pedidos `ENTREGUE` ou `CANCELADO` **não podem mais
ser alterados** (bloqueia todas as rotas de PATCH/itens).
