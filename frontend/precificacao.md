---
layout: default
title: Precificação
parent: Frontend
nav_order: 7
---

# Precificação

- Não existem mais faixas de preço mínimo/premium (nem os inputs de
  margem mínima/premium) — o cálculo só produz preço ideal para a
  margem desejada informada. O conceito de "status do preço praticado"
  (abaixo/dentro/acima do ideal) também foi removido por completo, junto
  com `StatusPrecoPraticadoBadge`.
- Listagem: para cada produto ativo, busca precificação em paralelo
  (`Promise.allSettled`) — produto sem cálculo fica com "-" na coluna de
  preço ideal. No lugar da antiga coluna de status, mostra quantos canais
  já têm preço praticado cadastrado (`GET /produtos/{id}/precos-canal`,
  também buscado em paralelo).
- Detalhe do produto: simulador "Comparar preço por canal" com
  **debounce de 400ms** e flag de cancelamento para evitar race
  condition; erro 400 sem produto com custo cadastrado mostra mensagem
  pedindo ficha técnica. Cada coluna de canal tem um campo **preço
  praticado** (obrigatório, deve ser > 0) e mostra o preço vigente já
  cadastrado para aquele canal (via `GET /produtos/{id}/precos-canal`),
  quando existir.
- "Confirmar este preço" por canal fica **desabilitado** enquanto o
  campo de preço praticado daquela coluna estiver vazio. Ao confirmar,
  dispara em sequência: `POST /precificacao/calcular` (mantém o
  histórico de simulação) e depois `PUT
  /produtos/{id}/precificacao/canal/{canalVendaId}/preco-praticado`
  (`{preco}`, upsert do preço vigente naquele canal); loading
  (`confirmandoCanalId`) cobre as duas chamadas. Falha em qualquer uma
  das duas mostra a mensagem de erro do backend (`ApiError.message`)
  abaixo do botão daquele canal especificamente.
- Histórico de cálculos não mostra mais preço mínimo/premium — só data,
  canal, margem desejada, preço ideal e preço praticado (campo legado da
  tabela de histórico, sem relação com o preço vigente por canal).
- Canais de venda: CRUD simples (lista + form reaproveitado); canal novo
  sempre nasce "ativo"; canais inativos aparecem com opacidade reduzida
  mas continuam editáveis.
