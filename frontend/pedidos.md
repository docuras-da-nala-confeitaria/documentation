---
layout: default
title: Pedidos
parent: Frontend
nav_order: 6
---

# Pedidos

- Novo pedido: carrega apenas canais `ATIVO`, pré-seleciona o primeiro
  automaticamente; itens montados localmente, enviados juntos no `POST
  /pedidos`.
- `SeletorItemPedido`: só produtos com status `ativo`; ao selecionar
  produto, **preenche automaticamente o valor unitário** com
  `preco_venda` (editável); valida produto+quantidade>0+valor≥0.
- Edição: `pedidoTravado` quando status é `entregue`/`cancelado` —
  desabilita formulário geral, seletor de status e itens.
- Opção "Entregue" no select fica `disabled` enquanto pagamento não está
  `pago`.
- **Confirmação obrigatória** (`window.confirm`) ao marcar como
  "Entregue", avisando explicitamente sobre baixa de estoque + lançamento
  de caixa automáticos.
- Valor total sempre exibido a partir do que a API retorna (sem
  recálculo local).
