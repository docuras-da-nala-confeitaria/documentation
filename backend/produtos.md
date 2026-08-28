---
layout: default
title: Produtos
parent: Backend
nav_order: 4
---

# Produtos (`/produtos`)

- `GET /produtos?categoria_id=&status=&modo_producao=` — filtros combináveis.
- `GET /produtos/{id}` — `id`, `nome`, `categoriaId`, `peso`, `status`,
  `modoProducao`. **Removido**: `precoVenda`, `custoAtual`,
  `lucroPorUnidade`, `margemLucroPercentual` não fazem mais parte do DTO —
  o preço de venda passou a ser por canal (ver `GET
  /produtos/{produtoId}/precos-canal` em [Precificação](precificacao.md)).
- `POST /produtos` — `nome`, `categoriaId` (deve existir), `peso`, `status`,
  `modoProducao`. **Removido**: `precoVenda` não é mais um campo do produto.
- `PATCH /produtos/{id}` — sem `@Valid`; substitui todos os campos.
- `PATCH /produtos/{id}/status` — ativa/inativa (não afeta pedidos/estoque
  existentes).

## Preço de venda por canal

Não existe mais um `precoVenda` único por produto — o preço praticado passa
a ser por combinação produto+canal, cadastrado via
[Precificação](precificacao.md) (`PUT
/produtos/{produtoId}/precificacao/canal/{canalVendaId}/preco-praticado`) e
consultado via `GET /produtos/{produtoId}/precos-canal` (lista `{
canalVendaId, canalVendaNome, preco, margem }`, uma linha por canal já
precificado).

## Canais de Venda (`/canais-venda`)

- `GET /canais-venda?status=` — filtro opcional `ATIVO`/`INATIVO`.
- `GET /canais-venda/{id}` — 404 se não achar.
- `POST /canais-venda` — `nome`, `taxaCanal`, `taxaPagamento`, `status`.
- `PATCH /canais-venda/{id}` — **se `taxaCanal` ou `taxaPagamento`
  mudarem**, dispara evento `CustoCanalVendaAtualizadoEvent` → recálculo
  automático de precificação de todos os produtos cujo **último** cálculo
  usou esse canal. Mudar apenas nome/status não recalcula nada.
