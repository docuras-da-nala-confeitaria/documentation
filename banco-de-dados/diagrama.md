---
layout: default
title: Diagrama
parent: Banco de Dados
nav_order: 2
---

# Diagrama de relacionamento

Redesenhado a partir das migrations Flyway atuais (`V1`–`V14`) — a imagem
`Diagrama_Relacionamento_Banco.png` original estava desatualizada (não
refletia, por exemplo, a estrutura de embalagem por múltiplos componentes
introduzida na `V11`, nem `canais_venda`/`recursos_producao`) e não foi
reaproveitada.

```mermaid
erDiagram
    usuarios ||--o{ pedidos : "registra"
    usuarios ||--o{ password_reset_tokens : "solicita"

    categorias ||--o{ produtos : "classifica"

    produtos ||--o| receitas : "tem"
    produtos ||--o| embalagens : "tem"
    produtos ||--o| produto_estoques : "tem"
    produtos ||--o{ produto_estoque_movimentacoes : "gera"
    produtos ||--o{ pedido_itens : "vendido em"
    produtos ||--o{ precificacoes : "calculado em"

    receitas ||--o{ receita_componentes : "compõe"
    receitas ||--o{ receita_recursos : "usa"
    receita_componentes ||--o{ receita_itens : "compõe"
    receita_itens }o--|| ingredientes : "consome"
    receita_recursos }o--|| recursos_producao : "usa"

    embalagens ||--o{ embalagem_componentes : "compõe"
    embalagem_componentes ||--o{ embalagem_itens : "compõe"
    embalagem_itens }o--|| ingredientes : "consome"

    ingredientes ||--o{ estoque_movimentacoes : "movimenta"
    ingredientes ||--o{ caixa_lancamentos : "origina (entrada de estoque)"

    canais_venda ||--o{ pedidos : "vendido via"
    canais_venda ||--o{ precificacoes : "calculado para"

    clientes ||--o{ pedidos : "faz"

    pedidos ||--o{ pedido_itens : "contém"
    pedidos ||--o{ estoque_movimentacoes : "gera (venda)"
    pedidos ||--o{ produto_estoque_movimentacoes : "gera (venda)"
    pedidos ||--o{ caixa_lancamentos : "origina"
```

## Notas de leitura

- `produtos` → `receitas` e `produtos` → `embalagens` são relações
  1:0..1 opcionais: um produto pode existir sem ficha técnica e/ou sem
  embalagem cadastrada.
- `embalagem_itens` e `receita_itens` referenciam a mesma tabela
  `ingredientes` — o campo `tipo` (`INGREDIENTE`/`EMBALAGEM`) em
  `ingredientes` é o que determina qual uso é válido em cada contexto
  (validado na aplicação, não por constraint de banco).
- `caixa_lancamentos.origem` determina se o lançamento veio de
  `pedido_id` (venda) ou `ingrediente_id` (entrada de estoque) — os dois
  campos são mutuamente exclusivos na prática, mas ambos nullable no
  schema.
- `metas` não tem relacionamento de FK com nenhuma outra tabela — é
  consultada por `mes_referencia` (string `yyyy-MM`) a partir do
  dashboard.

Para os campos e tipos completos de cada tabela, ver
[Modelo de dados](modelo-de-dados.md).
