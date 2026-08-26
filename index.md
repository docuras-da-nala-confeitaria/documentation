---
layout: default
title: Início
nav_order: 1
description: "Documentação técnica do sistema de gestão da Docuras da Nalá"
permalink: /
---

# Documentação — Sistema de Gestão Docuras da Nalá

Este site documenta o sistema de gestão da confeitaria: cadastros, estoque,
produção, pedidos, precificação e fluxo de caixa. O conteúdo é derivado
diretamente da leitura do código-fonte atual (backend Spring Boot + frontend
Next.js), das migrations do banco de dados e da infraestrutura de deploy.

## Seções

- **[Backend](backend/)** — rotas da API REST e regras de negócio implementadas no servidor.
- **[Frontend](frontend/)** — telas, fluxos e comportamentos implementados no cliente.
- **[Banco de Dados](banco-de-dados/)** — schema atual, reconstruído a partir das migrations.
- **[Arquitetura](arquitetura/)** — visão geral dos ambientes AWS (produção) e Raspberry Pi (local/on-premises).

## Como este conteúdo evolui

Não há versionamento separado do site — o histórico de commits/PRs deste
repositório é o próprio changelog da documentação. Alterações no sistema
devem vir acompanhadas de uma atualização correspondente nos arquivos `.md`
relevantes, no mesmo espírito de um PR normal de código.
