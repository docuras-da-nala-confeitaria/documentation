---
layout: default
title: Produtos
parent: Frontend
nav_order: 4
---

# Produtos

- Filtros: categoria, status, modo de produção, busca (client-side).
- Ação "Registrar produção" só aparece para produtos com `modo_producao
  !== "sob_encomenda"`.
- Página de edição com 4 blocos independentes (dados, receita+custo,
  embalagem+custo) — 404 de receita/embalagem é tratado como estado
  válido ("cadastre a ficha técnica"), não como erro.
- `FormProduto`: categoria pode ser criada inline (`POST /categorias`);
  peso deve ser > 0; **confirmação obrigatória** (`window.confirm`) ao
  inativar produto, com rollback se cancelado.
- `EditorReceita`/`EditorEmbalagem`: edição 100% local até salvar via
  `PUT`; validações client completas (rendimento/peso > 0, ao menos 1
  componente, nome/quantidade/tempo obrigatórios e > 0 em cada item).
- `ModalRegistrarProducao`: pré-preenche quantidade com o rendimento e
  validade com hoje; valida quantidade inteira > 0 e validade não
  anterior a hoje; 404 de receita mostra aviso específico com link para
  cadastrar ficha técnica.
