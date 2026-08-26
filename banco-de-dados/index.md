---
layout: default
title: Banco de Dados
nav_order: 4
has_children: true
permalink: /banco-de-dados/
---

# Banco de Dados

PostgreSQL. O schema é gerenciado por migrations **Flyway**
(`sistemas/backend/src/main/resources/db/migration/`, `V1__init.sql` até
`V14__data_entrega_pedido.sql` na data desta documentação).

Esta seção documenta o **estado final** do schema após todas as
migrations aplicadas em ordem — não o histórico migration a migration.
Sempre que uma nova migration for adicionada ao backend, esta página deve
ser atualizada para refletir o schema resultante.

Ver também: [Modelo de dados](modelo-de-dados.md) (tabelas e colunas) e
[Diagrama](diagrama.md) (relacionamento entre entidades).
