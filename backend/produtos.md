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

## Cardápio dinâmico (`GET /produtos/cardapio`)

- `GET /produtos/cardapio?canal_venda_id=&categoria_id=` — gera e devolve
  uma imagem PNG do cardápio (`Content-Type: image/png`), montada a
  partir dos produtos ativos com preço cadastrado no canal informado,
  agrupados por categoria.
- `canal_venda_id` — obrigatório. 404 se o canal não existe ou está
  inativo.
- `categoria_id` — opcional, repetível
  (`?categoria_id=A&categoria_id=B`). Restringe o cardápio a essas
  categorias; omitido = todas as categorias. Id de categoria que não
  existe é ignorado (sem erro).
- Produto ativo **sem** preço cadastrado nesse canal específico fica
  oculto do cardápio (não aparece como linha em branco).
- Internamente, o endpoint repassa a chamada para o `cardapio-service`
  (serviço Python interno, não exposto à internet, que desenha a imagem
  com Pillow e lê o banco direto) — 502 se esse serviço estiver fora do
  ar ou responder algo inesperado.

## Categorias de produto

- `GET /categorias` — lista todas as categorias.
- `POST /categorias` — cria uma categoria (`{ "nome": "..." }`).
- `PUT /categorias/{id}` — renomeia uma categoria.
- `DELETE /categorias/{id}` — exclui uma categoria. Só funciona se
  **nenhum produto** estiver vinculado a ela (`produtos.categoria_id`);
  caso contrário devolve `409` com uma mensagem informando quantos
  produtos ainda usam essa categoria. Não desvincula nem exclui produtos
  automaticamente — é preciso mudar a categoria desses produtos (ou
  excluí-los) antes de conseguir excluir a categoria.
