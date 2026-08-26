---
layout: default
title: Padrões transversais
parent: Frontend
nav_order: 10
---

# Padrões transversais do frontend

1. Sem lib de data-fetching — `useState`/`useEffect`/`useCallback`
   manuais replicados em cada página.
2. Enums normalizados para minúsculas no client, convertidos para
   maiúsculas ao enviar ao backend.
3. Erros de API repassam `err.message` do backend quando disponível.
4. **401 → logout automático** em qualquer chamada autenticada.
5. Confirmações via `window.confirm` para ações sensíveis: inativar
   produto, marcar pedido como entregue, remover item de pedido, excluir
   lançamento manual de caixa.
6. Skeletons enquanto dado é `null` (convenção: `null` = não carregado,
   array vazio = carregado e vazio).
7. Ordenação alfabética client-side (`localeCompare` pt-BR) em várias
   listagens — o backend não garante ordem.
8. Sem geração de PDF, impressão ou upload de imagem em nenhuma tela.
9. Sem paginação real de UI — listas usam `limit=500`.
10. Proteção contra race conditions via contador de sequência (`useRef`)
    em Caixa e Movimentações de Produto.
11. Debounce só existe no simulador de comparação de preços por canal;
    buscas de clientes/produtos disparam uma chamada por tecla.
