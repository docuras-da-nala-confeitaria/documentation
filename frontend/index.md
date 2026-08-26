---
layout: default
title: Frontend
nav_order: 3
has_children: true
permalink: /frontend/
---

# Frontend (Next.js / React)

Stack: React 19 + Next.js App Router (grupo `(app)` autenticado), quase
tudo client component. Tailwind, Recharts. Sem React Query/SWR — chamadas
via `fetch` encapsulado em `lib/api.ts`.

## Infraestrutura compartilhada (`lib/`)

**`lib/api.ts`** — `apiFetch<T>` monta requisições para
`NEXT_PUBLIC_API_URL`, injeta `Authorization: Bearer` quando há token;
erros viram `ApiError(status, message)` (mensagem lida do corpo JSON, ou
genérica se ausente); 204 retorna `undefined`.

**`lib/auth-context.tsx`** — `AuthProvider` guarda
`usuario`/`token`/`carregando`. Restaura sessão do
`localStorage["confeitaria.token"]` via `GET /auth/me` ao montar.
`login`/`logout` gerenciam o token. `apiFetchAutenticado`: em **401, força
logout automaticamente** (interceptor de sessão expirada). O campo `papel`
existe no modelo `Usuario` mas **nenhuma tela usa isso para restringir
ações** — sem autorização por papel no client.

**`lib/format.ts`** — `formatarMoeda` (Intl BRL), `formatarDataCurta`
(parse manual de ISO para evitar bug de timezone), `normalizarEnum`
(maiúsculas do backend → minúsculas internas).

**`lib/types.ts`** — contrato de dados completo (Usuario, CanalVenda,
Ingrediente, Produto/Receita/Embalagem, Cliente, Pedido, Precificacao,
CaixaLancamento) e union types de todos os enums.

**`lib/*-labels.ts`** — mapas enum → rótulo em português usados em
badges/selects.

Ver também [Padrões transversais](padroes-transversais.md).
