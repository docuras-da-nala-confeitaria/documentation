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
- O valor unitário de um item não vem mais de um `preco_venda` fixo do
  produto — vem do **preço praticado daquele produto no canal
  selecionado do pedido** (`GET
  /produtos/{id}/precificacao/canal/{canalVendaId}/preco`). Por isso o
  seletor de itens fica desabilitado até haver canal escolhido.
- `SeletorItemPedido`: só produtos com status `ativo`; recebe
  `canalVendaId` do pedido; ao selecionar um produto, busca o preço
  vigente naquele canal e **preenche automaticamente o valor unitário**
  (editável); se a API retornar erro (produto sem preço cadastrado
  naquele canal), mostra a mensagem do backend (`ApiError.message`) e
  não deixa adicionar o item; valida produto+quantidade>0+valor≥0.
- Ao trocar o canal de venda **enquanto o pedido novo ainda está sendo
  montado** (antes de criar), todos os itens já adicionados têm o valor
  unitário **recalculado automaticamente** para o novo canal (uma
  chamada por item); item sem preço no novo canal mantém o valor
  anterior e é listado num aviso, bloqueando a criação do pedido até ser
  removido ou precificado naquele canal.
- Em um pedido já criado (`/pedidos/[id]`), o **canal de venda não é
  editável** — aparece como texto somente leitura; novos itens buscam o
  preço pelo canal fixo do pedido (`pedido.canal_venda_id`), sem opção
  de recálculo por outro canal.
- Edição: `pedidoTravado` quando status é `entregue`/`cancelado` —
  desabilita formulário geral, seletor de status e itens.
- Opção "Entregue" no select fica `disabled` enquanto pagamento não está
  `pago`.
- **Confirmação obrigatória** (`window.confirm`) ao marcar como
  "Entregue", avisando explicitamente sobre baixa de estoque + lançamento
  de caixa automáticos.
- Valor total sempre exibido a partir do que a API retorna (sem
  recálculo local).
