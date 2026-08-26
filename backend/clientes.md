---
layout: default
title: Clientes
parent: Backend
nav_order: 3
---

# Clientes (`/clientes`)

- `GET /clientes?busca=` — sem `busca` lista todos; com `busca` filtra por
  nome OU telefone (contains, case-insensitive para nome).
- `POST /clientes` — `nome`, `telefone`, `endereco`; sem regra de unicidade.
- `PATCH /clientes/{id}` — sem `@Valid`; sobrescreve sempre os três campos
  (não é patch parcial de verdade).

> Não existe `GET /clientes/{id}` no backend (o frontend contorna isso
> filtrando a lista completa).

## Categorias (`/categorias`)

- `GET /categorias` — lista todas.
- `POST /categorias` — cria (`nome`), sem checagem de duplicidade.
