---
layout: default
title: Estoque
parent: Frontend
nav_order: 5
---

# Estoque

- Duas abas: Ingredientes (`GET /ingredientes?tipo=&status=`) e Produtos
  Prontos (`GET /estoque-produtos`, carregado sob demanda).
- **`FormEntrada` com lógica dupla por unidade de medida**:
  - Unidade `un`: quantidade + custo unitário, com **recálculo
    automático** `custo_unitario = custo_total / quantidade` (editável
    depois).
  - Unidades `g`/`ml`: formulário "por embalagem" (nº de embalagens ×
    quantidade por embalagem × fator de conversão kg/l→1000), calcula
    quantidade total e custo unitário em tempo real antes de enviar.
- `ModalRegistrarDescarte`: quantidade deve ser inteiro entre 1 e a
  quantidade atual (validado no client).
- Movimentações de produto: usa `sequenciaRef` para descartar respostas de
  filtro desatualizadas (proteção contra race condition).
