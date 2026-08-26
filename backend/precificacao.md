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

- Input: `margemDesejada` (obrigatória), `margemMinima` (default = desejada
  − 15), `margemPremium` (default = desejada + 15), `custoProducao`
  (opcional, override manual), `perdasPercentual` (opcional, default 0),
  `canalVendaId` (**obrigatório**, sem override manual de taxa).
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
  8. Preço por faixa (markup sobre preço, não sobre custo): `preco =
     custoTotal / (1 - margem/100)`; margem ≥ 100% gera erro.
  9. Calcula preço mínimo/ideal/premium para as três margens.
- **Cada cálculo gera novo registro de histórico** — nunca sobrescreve o
  anterior.

- `GET /produtos/{produtoId}/precificacao/comparar-canais?margem_desejada=&margem_minima=&margem_premium=`
  — calcula preço para todos os canais ativos, **sem persistir** (evita
  poluir o histórico).
- `GET /produtos/{produtoId}/precificacao/historico` — todo o histórico do
  produto.
- `PATCH /produtos/{produtoId}/precificacao/praticado` — registra preço
  realmente cobrado **sobre o cálculo mais recente**, sem criar novo
  histórico. Status calculado dinamicamente (tolerância de 5%):
  `ABAIXO_DO_IDEAL` / `DENTRO_DO_IDEAL` / `ACIMA_DO_IDEAL`.

**Recálculo automático (efeito colateral, não é rota)**: disparado por
mudança de custo de ingrediente, custo/hora de recurso, ou taxas de canal.
Reaproveita parâmetros do último cálculo de cada produto afetado; produtos
sem cálculo prévio são ignorados. Para canal, só recalcula produtos cujo
último cálculo usou aquele canal específico.
