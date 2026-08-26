---
layout: default
title: Estoque de Produtos e Produção
parent: Backend
nav_order: 8
---

# Estoque de Produtos Prontos / Produção

- `GET /estoque-produtos` — visão geral de todos os produtos já produzidos
  alguma vez.
- `GET /estoque-produtos/movimentacoes?dataInicio=&dataFim=` —
  movimentações de todos os produtos.
- `GET /produtos/{produtoId}/estoque` — se nunca produzido, retorna DTO
  zerado sem persistir nada.
- `GET /produtos/{produtoId}/estoque/movimentacoes?dataInicio=&dataFim=` —
  histórico do produto.

## `POST /produtos/{produtoId}/producao`

`quantidade` (>0), `validade` (opcional). Regras críticas:

- Só permitido para produtos com `modoProducao != SOB_ENCOMENDA`.
- Receita deve ter ao menos um ingrediente.
- Calcula fator de escala = `quantidadeProduzida / rendimento`.
- **Validação em duas passagens**: soma consumo total necessário por
  ingrediente (agregando componentes) e só aplica baixas se **todos**
  tiverem estoque suficiente (evita baixa parcial); erro lista tudo que
  falta.
- Dá baixa nos ingredientes, recalcula status, registra
  `EstoqueMovimentacao SAIDA` para cada um.
- Soma quantidade produzida ao `ProdutoEstoque.quantidadeDisponivel` (cria
  se não existir); atualiza validade se informada.
- Registra `ProdutoEstoqueMovimentacao PRODUCAO`.
- Retorna custo total/unitário de produção. Não gera lançamento de caixa
  nem recálculo de precificação (produção não é venda).
