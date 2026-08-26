---
layout: default
title: Estoque de Insumos
parent: Backend
nav_order: 7
---

# Ingredientes / Estoque de Insumos

- `GET /ingredientes?tipo=&status=` — filtros por tipo e `StatusEstoque`.
- `GET /ingredientes/{id}` — busca por id.
- `POST /ingredientes` — `nome` (único, case-insensitive), `tipo`,
  `unidadeMedida`, `estoqueMinimo`, `custoUnitario`; `quantidadeAtual`
  sempre nasce **zero**.
- `PATCH /ingredientes/{id}` — sem `@Valid`; não altera `quantidadeAtual`
  diretamente. **Se `custoUnitario` mudar**, dispara
  `CustoIngredienteAtualizadoEvent` → recálculo automático de preços dos
  produtos que usam esse ingrediente (reaproveitando parâmetros do último
  cálculo de cada produto).

**Regra de status de estoque** (`recalcularStatus`):

- `estoqueMinimo` nulo/zero → sempre `SUFICIENTE`.
- `quantidadeAtual <= estoqueMinimo` → `BAIXO`.
- `quantidadeAtual <= estoqueMinimo × 1.2` → `COMPRAR_EM_BREVE`.
- Caso contrário → `SUFICIENTE`.
- Recalculado a cada entrada, descarte, venda, produção ou edição de
  cadastro.

- `POST /ingredientes/{id}/entrada` — `quantidade`, `custoUnitario`, `data`
  (opcional, default hoje). Soma ao estoque; **sobrescreve
  `custoUnitario`** pelo valor da compra mais recente (não é custo médio);
  recalcula status; registra movimentação `ENTRADA`. Se o custo mudou,
  dispara recálculo de precificação. **Efeito colateral em Caixa**: cria
  automaticamente lançamento `SAIDA`, categoria "Ingredientes", valor =
  `quantidade × custoUnitario`, origem `ENTRADA_ESTOQUE` (não editável
  manualmente depois).
- `POST /ingredientes/{id}/descarte` — `quantidade`, `motivo`,
  `observacao`; quantidade não pode exceder o estoque atual; reduz
  `quantidadeAtual`, recalcula status, registra movimentação `DESCARTE`.
  Não gera lançamento de caixa nem altera custo.
- `GET /ingredientes/{id}/movimentacoes?dataInicio=&dataFim=` — histórico
  do ingrediente.
- `GET /estoque/alertas` — lista ingredientes com status
  `COMPRAR_EM_BREVE` ou `BAIXO`.
- `GET /estoque/movimentacoes?dataInicio=&dataFim=` (obrigatórios) — todas
  as movimentações de qualquer ingrediente no período.
