---
layout: default
title: Receitas e Embalagens
parent: Backend
nav_order: 5
---

# Receitas (Ficha Técnica)

- `GET /produtos/{produtoId}/receita` — 404 se produto não tem receita.
- `PUT /produtos/{produtoId}/receita` — **substituição total**: cria se não
  existir, ou apaga e recria por completo componentes/itens/recursos a
  partir do input (não é merge incremental). Cada ingrediente/recurso
  referenciado deve existir.
- `POST /produtos/{produtoId}/receita/componentes` — adiciona componente
  vazio (404 se receita ainda não existe).
- `GET /produtos/{produtoId}/receita/custo` — `custoTotal` (soma
  `quantidadeUsada × custoUnitario` de todos ingredientes), `custoPorUnidade
  = custoTotal / rendimento`, `custoProducaoTotal`/`custoProducaoPorUnidade`
  (baseado em `tempoMinutos × custoPorHora` dos recursos usados).
- `PATCH /receita-componentes/{id}` — renomeia componente.
- `DELETE /receita-componentes/{id}` — remove componente (cascata para
  itens).
- `POST /receita-componentes/{id}/itens` — adiciona item (`ingredienteId`,
  `quantidadeUsada`).
- `PATCH /receita-itens/{id}` — atualiza quantidade.
- `DELETE /receita-itens/{id}` — remove item.

# Embalagens

- `GET /produtos/{produtoId}/embalagem` — embalagem é **opcional**: se não
  existir, retorna DTO vazio (`id=null`), não 404.
- `PUT /produtos/{produtoId}/embalagem` — substituição total (mesmo padrão
  de Receita). Cada ingrediente usado deve ter `tipo == EMBALAGEM`.
  - **Categorias de componente**: `POR_UNIDADE` (escala com a quantidade
    vendida, ex.: saquinho) vs. `POR_VENDA` (consumido **uma vez por
    pedido**, ex.: sacola de entrega).
- `POST /produtos/{produtoId}/embalagem/componentes` — adiciona componente
  vazio (cria embalagem se não existir).
- `GET /produtos/{produtoId}/embalagem/custo` — `custoPorUnidade` (soma
  itens POR_UNIDADE), `custoPorVenda` (soma itens POR_VENDA), `custoTotal =
  custoPorUnidade × rendimento` (ou `null` sem receita). Os dois totais
  nunca se somam entre si.
