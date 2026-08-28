---
layout: default
title: Precificação
parent: Backend
nav_order: 10
---

# Precificação

- `GET /produtos/{produtoId}/precificacao` — retorna cálculo mais recente;
  404 se nunca calculado.

## `POST /produtos/{produtoId}/precificacao/calcular` — coração do módulo

- Input: `margemDesejada` (obrigatória), `custoProducao` (opcional, override
  manual), `perdasPercentual` (opcional, default 0), `canalVendaId`
  (**obrigatório**, sem override manual de taxa). `margemMinima`/
  `margemPremium` ainda são aceitos no body por compatibilidade, mas são
  **ignorados** — não geram mais nenhum valor.
- Fórmula:
  1. `custoReceitaPorUnidade = custoTotalReceita / rendimento`.
  2. `custoEmbalagem` = soma bruta dos itens `POR_UNIDADE` (itens
     `POR_VENDA` não entram aqui).
  3. `custoProducao` = valor manual OU calculado (`custoPorHora ×
     tempoMinutos` dos recursos / rendimento).
  4. `subtotal = custoReceitaPorUnidade + custoEmbalagem + custoProducao`.
  5. `taxasPercentual = canal.taxaCanal + canal.taxaPagamento`.
  6. `perdasValor = subtotal × perdas%`; `taxasValor = subtotal × taxas%`.
  7. `custoTotal = subtotal + perdasValor + taxasValor`.
  8. `precoIdeal = custoTotal / (1 - margemDesejada/100)`; margem ≥ 100%
     gera erro.
- **Cada cálculo gera novo registro de histórico** — nunca sobrescreve o
  anterior.
- **Removido**: as faixas `precoMinimo`/`precoPremium` (e
  `margemMinima`/`margemPremium`) deixaram de ser calculadas — a resposta
  só traz `precoIdeal`. As colunas continuam existindo na tabela
  `precificacoes` por compatibilidade histórica, mas ficam sempre `null`
  em registros novos.
- **Removido**: o enum/status `StatusPrecoPraticado`
  (`ABAIXO_DO_IDEAL`/`DENTRO_DO_IDEAL`/`ACIMA_DO_IDEAL`) foi eliminado por
  completo — não é mais calculado nem exposto na resposta.

- `GET /produtos/{produtoId}/precificacao/comparar-canais?margem_desejada=&margem_minima=&margem_premium=`
  — calcula preço para todos os canais ativos, **sem persistir** (evita
  poluir o histórico). `margem_minima`/`margem_premium` continuam aceitos
  na query string por compatibilidade, mas ignorados.
- `GET /produtos/{produtoId}/precificacao/historico` — todo o histórico do
  produto.

## Preço praticado por canal (tabela dedicada `produto_preco_canal`)

O preço vigente que a confeitaria realmente cobra por um produto em cada
canal não mora mais só no histórico de cálculos. Ele mora numa tabela
dedicada, `produto_preco_canal` (`produto_id`, `canal_venda_id`, `preco`,
`atualizado_em`, único por produto+canal — só o valor **atual**, sem
histórico de mudanças), justamente para não ser apagado silenciosamente
quando um recálculo automático de custo gera uma nova linha de histórico
sem preço praticado.

Para preservar o rastro de "quanto foi praticado a cada confirmação" (já
que a tabela nova só guarda o valor vigente), confirmar um preço também
grava o mesmo valor em `precificacoes.preco_praticado` do cálculo mais
recente daquele produto+canal — por isso a coluna "Praticado" do histórico
de precificação continua sendo preenchida a cada confirmação.

- `PUT /produtos/{produtoId}/precificacao/canal/{canalVendaId}/preco-praticado`
  — body `{ "preco": <valor> }`, **obrigatório** (400 se ausente/nulo).
  Responde **204 No Content**. Faz **upsert** em `produto_preco_canal`
  (atualiza o preço vigente se já existir um registro para aquele
  produto+canal, cria se não existir) e também grava `preco_praticado` no
  cálculo mais recente do produto+canal em `precificacoes` (se existir
  algum). Substitui a antiga rota `PATCH /produtos/{id}/precificacao/praticado`
  (removida).
- `GET /produtos/{produtoId}/precificacao/canal/{canalVendaId}/preco` —
  retorna `{ "preco": <valor> }` vigente daquele produto naquele canal.
  **404** (`RecursoNaoEncontradoException`) se nunca foi confirmado um
  preço para essa combinação — orienta o usuário a calcular/confirmar a
  precificação antes de usar o produto num pedido nesse canal.
- `GET /produtos/{produtoId}/precos-canal` — lista `{ canalVendaId,
  canalVendaNome, preco, margem }`, uma linha por canal já precificado
  para o produto. `margem = (preco − custoAtual) / preco × 100`, `null` se
  o produto não tem receita (sem `custoAtual`) ou se `preco` é zero.

**Recálculo automático (efeito colateral, não é rota)**: disparado por
mudança de custo de ingrediente, custo/hora de recurso, ou taxas de canal.
Reaproveita parâmetros do último cálculo de cada produto afetado; produtos
sem cálculo prévio são ignorados. Para canal, só recalcula produtos cujo
último cálculo usou aquele canal específico. Como o preço praticado agora
mora em `produto_preco_canal` (tabela separada do histórico), esse
recálculo automático não apaga mais o preço vigente confirmado.
