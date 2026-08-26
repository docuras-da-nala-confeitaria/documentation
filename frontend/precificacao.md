---
layout: default
title: Precificação
parent: Frontend
nav_order: 7
---

# Precificação

- Listagem: para cada produto ativo, busca precificação em paralelo
  (`Promise.allSettled`) — produto sem cálculo fica com "-".
- Detalhe do produto: simulador "Comparar preço por canal" com
  **debounce de 400ms** e flag de cancelamento para evitar race
  condition; erro 400 sem produto com custo cadastrado mostra mensagem
  pedindo ficha técnica.
- "Confirmar este preço" por canal: `POST /precificacao/calcular`
  persiste o cálculo daquele canal como oficial; loading independente por
  canal.
- Canais de venda: CRUD simples (lista + form reaproveitado); canal novo
  sempre nasce "ativo"; canais inativos aparecem com opacidade reduzida
  mas continuam editáveis.
