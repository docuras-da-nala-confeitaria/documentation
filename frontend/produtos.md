---
layout: default
title: Produtos
parent: Frontend
nav_order: 4
---

# Produtos

- Produto não tem mais um "preço de venda" único (`preco_venda` foi
  removido, junto com `custo_atual`/`lucro_por_unidade`/
  `margem_lucro_percentual` do DTO) — o preço de venda passou a
  depender do canal de venda, ver Precificação.
- Listagem simplificada: só **Nome, Categoria, Peso, Status, Modo de
  produção, Ações** — nenhuma coluna de preço/custo/margem.
- Filtros: categoria, status, modo de produção, busca (client-side).
- Ação "Registrar produção" só aparece para produtos com `modo_producao
  !== "sob_encomenda"`.
- Página de edição com 4 blocos independentes (dados, **preço praticado
  por canal**, receita+custo, embalagem+custo) — 404 de receita/embalagem
  é tratado como estado válido ("cadastre a ficha técnica"), não como
  erro.
- No lugar do antigo campo "Preço de venda", a página de edição mostra
  uma tabela somente-leitura "Preço praticado por canal" (Canal de
  venda / Preço praticado / Margem), vinda de `GET
  /produtos/{id}/precos-canal` — só lista canais que já têm preço
  cadastrado; produto sem nenhum preço mostra mensagem indicando para
  calcular na tela de Precificação. Editar o preço continua sendo feito
  lá, não nesta página.
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
