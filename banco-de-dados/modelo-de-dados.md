---
layout: default
title: Modelo de dados
parent: Banco de Dados
nav_order: 1
---

# Modelo de dados

Todas as tabelas usam `id UUID PRIMARY KEY DEFAULT gen_random_uuid()`,
salvo indicação em contrário. Reconstruído a partir das migrations Flyway
`V1` a `V14`.

## usuarios

| Coluna | Tipo | Observações |
|---|---|---|
| nome | VARCHAR(255) NOT NULL | |
| email | VARCHAR(255) NOT NULL UNIQUE | |
| senha_hash | VARCHAR(255) NOT NULL | BCrypt |
| papel | VARCHAR(20) NOT NULL | `ADMIN` / `FUNCIONARIO` |
| ativo | BOOLEAN NOT NULL DEFAULT TRUE | |

## categorias

| Coluna | Tipo |
|---|---|
| nome | VARCHAR(255) NOT NULL |

## ingredientes

| Coluna | Tipo | Observações |
|---|---|---|
| nome | VARCHAR(255) NOT NULL | único case-insensitive (`idx_ingredientes_nome_unico` sobre `LOWER(nome)`) |
| tipo | VARCHAR(20) NOT NULL | `INGREDIENTE` / `EMBALAGEM` |
| unidade_medida | VARCHAR(20) NOT NULL | |
| quantidade_atual | NUMERIC(14,5) NOT NULL DEFAULT 0 | |
| estoque_minimo | NUMERIC(14,5) NOT NULL | |
| custo_unitario | NUMERIC(14,5) NOT NULL | |
| status | VARCHAR(20) NOT NULL DEFAULT 'SUFICIENTE' | |

## produtos

| Coluna | Tipo | Observações |
|---|---|---|
| nome | VARCHAR(255) NOT NULL | |
| categoria_id | UUID NOT NULL REFERENCES categorias | |
| peso | NUMERIC(14,4) NOT NULL | |
| preco_venda | NUMERIC(14,2) | |
| status | VARCHAR(20) NOT NULL DEFAULT 'ATIVO' | |
| modo_producao | VARCHAR(20) NOT NULL | |

> A coluna `embalagem_id` existia no schema original (`V1`) mas foi
> removida na `V11`, substituída pela tabela `embalagens` (múltiplos
> componentes).

## receitas

Renomeada de `fichas_tecnicas` (`V3`).

| Coluna | Tipo |
|---|---|
| produto_id | UUID NOT NULL UNIQUE REFERENCES produtos |
| rendimento | INTEGER NOT NULL |
| peso_unidade | NUMERIC(14,4) NOT NULL |

## receita_componentes

Renomeada de `ficha_componentes` (`V3`).

| Coluna | Tipo |
|---|---|
| receita_id | UUID NOT NULL REFERENCES receitas |
| nome_componente | VARCHAR(255) NOT NULL |

## receita_itens

Renomeada de `ficha_itens` (`V3`).

| Coluna | Tipo |
|---|---|
| componente_id | UUID NOT NULL REFERENCES receita_componentes |
| ingrediente_id | UUID NOT NULL REFERENCES ingredientes |
| quantidade_usada | NUMERIC(14,5) NOT NULL |

## recursos_producao (`V12`)

| Coluna | Tipo |
|---|---|
| nome | VARCHAR(255) NOT NULL |
| custo_por_hora | NUMERIC(14,4) NOT NULL |

## receita_recursos (`V12`)

Relaciona uma receita a recursos de produção usados.

| Coluna | Tipo |
|---|---|
| receita_id | UUID NOT NULL REFERENCES receitas |
| recurso_id | UUID NOT NULL REFERENCES recursos_producao |
| tempo_minutos | NUMERIC(14,2) NOT NULL |

## embalagens (`V11`)

| Coluna | Tipo |
|---|---|
| produto_id | UUID NOT NULL UNIQUE REFERENCES produtos |

## embalagem_componentes (`V11`)

| Coluna | Tipo |
|---|---|
| embalagem_id | UUID NOT NULL REFERENCES embalagens |
| nome_componente | VARCHAR(255) NOT NULL |
| categoria | VARCHAR(20) NOT NULL — `POR_UNIDADE` / `POR_VENDA` |

## embalagem_itens (`V11`)

| Coluna | Tipo |
|---|---|
| componente_id | UUID NOT NULL REFERENCES embalagem_componentes |
| ingrediente_id | UUID NOT NULL REFERENCES ingredientes |
| quantidade_usada | NUMERIC(14,5) NOT NULL |

## canais_venda (`V13`)

Substitui o antigo enum fixo `canal_venda` em `pedidos`.

| Coluna | Tipo |
|---|---|
| nome | VARCHAR(50) NOT NULL |
| taxa_canal | NUMERIC(14,2) NOT NULL DEFAULT 0 |
| taxa_pagamento | NUMERIC(14,2) NOT NULL DEFAULT 0 |
| status | VARCHAR(20) NOT NULL DEFAULT 'ATIVO' |

## precificacoes

| Coluna | Tipo | Observações |
|---|---|---|
| produto_id | UUID NOT NULL REFERENCES produtos | |
| custo_total | NUMERIC(14,2) NOT NULL | |
| margem_desejada | NUMERIC(14,2) NOT NULL | |
| margem_minima | NUMERIC(14,2) | `V5` |
| margem_premium | NUMERIC(14,2) | `V5` |
| preco_minimo | NUMERIC(14,2) | |
| preco_ideal | NUMERIC(14,2) | |
| preco_premium | NUMERIC(14,2) | |
| preco_praticado | NUMERIC(14,2) | |
| custo_producao | NUMERIC(14,2) | `V12`, substitui `gas_energia`/`mao_de_obra` |
| custo_producao_manual | BOOLEAN NOT NULL DEFAULT FALSE | `V12` |
| perdas_percentual | NUMERIC(14,2) | |
| taxas_percentual | NUMERIC(14,2) | |
| canal_venda_id | UUID REFERENCES canais_venda | `V13`, nullable |
| data_calculo | DATE NOT NULL DEFAULT CURRENT_DATE | |
| criado_em | TIMESTAMP NOT NULL DEFAULT now() | `V7`, desempate de ordenação quando há mais de um cálculo no mesmo dia |

> `gas_energia` e `mao_de_obra` (colunas originais do `V1`) foram
> removidas na `V12`, sem migração de dados históricos.

## clientes

| Coluna | Tipo |
|---|---|
| nome | VARCHAR(255) NOT NULL |
| telefone | VARCHAR(30) NOT NULL |
| endereco | VARCHAR(500) |

## pedidos

| Coluna | Tipo | Observações |
|---|---|---|
| numero | SERIAL UNIQUE | sequencial automático |
| data | DATE NOT NULL DEFAULT CURRENT_DATE | |
| cliente_id | UUID NOT NULL REFERENCES clientes | |
| usuario_id | UUID REFERENCES usuarios | nullable |
| canal_venda_id | UUID NOT NULL REFERENCES canais_venda | `V13`, substitui coluna `canal_venda` (enum fixo) |
| forma_pagamento | VARCHAR(50) NOT NULL | |
| taxa_entrega | NUMERIC(14,2) DEFAULT 0 | |
| desconto | NUMERIC(14,2) DEFAULT 0 | |
| status_pagamento | VARCHAR(20) NOT NULL DEFAULT 'PENDENTE' | |
| status_pedido | VARCHAR(20) NOT NULL DEFAULT 'EM_PRODUCAO' | |
| horario_entrega | VARCHAR(50) | texto livre |
| data_entrega | DATE | `V14`, nullable |
| observacoes | VARCHAR(1000) | |

## pedido_itens

| Coluna | Tipo |
|---|---|
| pedido_id | UUID NOT NULL REFERENCES pedidos |
| produto_id | UUID NOT NULL REFERENCES produtos |
| quantidade | INTEGER NOT NULL |
| valor_unitario | NUMERIC(14,2) NOT NULL |

## estoque_movimentacoes

| Coluna | Tipo | Observações |
|---|---|---|
| ingrediente_id | UUID NOT NULL REFERENCES ingredientes | |
| tipo | VARCHAR(20) NOT NULL | `ENTRADA` / `SAIDA` / `DESCARTE` |
| quantidade | NUMERIC(14,5) NOT NULL | |
| custo_unitario | NUMERIC(14,5) | |
| data | DATE NOT NULL DEFAULT CURRENT_DATE | |
| pedido_id | UUID REFERENCES pedidos | nullable |
| motivo | VARCHAR(20) | `V10`, usado no descarte |
| observacao | TEXT | `V10` |

## produto_estoques

| Coluna | Tipo |
|---|---|
| produto_id | UUID NOT NULL UNIQUE REFERENCES produtos |
| quantidade_disponivel | INTEGER NOT NULL DEFAULT 0 |
| validade | DATE |
| atualizado_em | TIMESTAMP NOT NULL DEFAULT now() |

## produto_estoque_movimentacoes

| Coluna | Tipo |
|---|---|
| produto_id | UUID NOT NULL REFERENCES produtos |
| tipo | VARCHAR(20) NOT NULL — `PRODUCAO` / `VENDA` |
| quantidade | INTEGER NOT NULL |
| data | DATE NOT NULL DEFAULT CURRENT_DATE |
| pedido_id | UUID REFERENCES pedidos, nullable |
| observacao | VARCHAR(500) |

## caixa_lancamentos

| Coluna | Tipo | Observações |
|---|---|---|
| tipo | VARCHAR(20) NOT NULL | `ENTRADA` / `SAIDA` |
| categoria | VARCHAR(100) NOT NULL | |
| valor | NUMERIC(14,2) NOT NULL | |
| data | DATE NOT NULL | |
| pedido_id | UUID REFERENCES pedidos | nullable |
| ingrediente_id | UUID REFERENCES ingredientes | `V9`, nullable |
| origem | VARCHAR(20) NOT NULL DEFAULT 'MANUAL' | `V9` — `MANUAL` / `PEDIDO` / `ENTRADA_ESTOQUE` |
| descricao | VARCHAR(500) | |

## metas

| Coluna | Tipo |
|---|---|
| mes_referencia | VARCHAR(7) NOT NULL UNIQUE |
| valor_meta | NUMERIC(14,2) NOT NULL |

## password_reset_tokens (`V8`)

| Coluna | Tipo |
|---|---|
| usuario_id | UUID NOT NULL REFERENCES usuarios |
| token | VARCHAR(255) NOT NULL UNIQUE |
| expira_em | TIMESTAMP NOT NULL |
| usado_em | TIMESTAMP |
| criado_em | TIMESTAMP NOT NULL DEFAULT now() |

## Tabelas removidas

- **taxas_canal_venda** (`V6`) — substituída por `canais_venda` (`V13`),
  que já inclui `taxa_canal`; dados migrados via seed antes do `DROP
  TABLE`.
