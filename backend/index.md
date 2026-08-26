---
layout: default
title: Backend
nav_order: 2
has_children: true
permalink: /backend/
---

# Backend (API REST)

Base package: `br.com.docurasdanala.gereciamento_confeitaria`. Todos os
controllers são `@RestController`. Não há prefixo global de contexto nas
rotas.

Stack: Spring Boot (Java), PostgreSQL, migrations via Flyway.

## Segurança (observação transversal)

Não há `SecurityFilterChain`, `@EnableWebSecurity` nem
`@PreAuthorize`/`hasRole` em nenhum controller do backend Java. O projeto:

- Gera e valida JWT (`JwtService`), mas **não há filtro que exija o token nas
  rotas do Spring** — a validação real do JWT em produção é feita fora do
  Spring, por um **Lambda Authorizer do API Gateway (AWS)**, usando a mesma
  chave de assinatura.
- Não há controle de autorização por papel dentro do código: `papel`
  (`ADMIN`/`FUNCIONARIO`) vai como claim no JWT, mas nenhum service/controller
  o utiliza para restringir acesso.
- CORS (`WebConfig`) libera GET/POST/PUT/PATCH/DELETE/OPTIONS para origens
  configuradas em `app.cors.allowed-origin-patterns` (padrão
  `localhost:3000`/`3001`, aceitando wildcard).
- Senhas com `BCryptPasswordEncoder`.

**Conclusão para documentação de segurança**: autenticação real ocorre fora
do código Java (API Gateway); não há autorização por papel implementada em
nenhuma camada do backend analisado.

## Enums principais (referência)

- `Papel`: `ADMIN`, `FUNCIONARIO` (só claim JWT, sem enforcement no backend).
- `StatusPedido`: `EM_PRODUCAO`, `A_CAMINHO`, `ENTREGUE`, `CANCELADO`.
- `StatusPagamento`: `PENDENTE`, `PAGO`.
- `StatusAtivoInativo`: `ATIVO`, `INATIVO` (Produto, CanalVenda).
- `StatusEstoque`: `SUFICIENTE`, `COMPRAR_EM_BREVE`, `BAIXO`.
- `TipoItemEstoque`: inclui `INGREDIENTE`, `EMBALAGEM`.
- `CategoriaEmbalagem`: `POR_UNIDADE`, `POR_VENDA`.
- `TipoLancamento`: `ENTRADA`, `SAIDA`.
- `OrigemLancamento`: `MANUAL`, `PEDIDO`, `ENTRADA_ESTOQUE`.
- `TipoMovimentacaoEstoque`: `ENTRADA`, `SAIDA`, `DESCARTE`.
- `TipoMovimentacaoProduto`: `PRODUCAO`, `VENDA` (e possivelmente `PERDA`/`AJUSTE`).
- `ModoProducao`: `SOB_ENCOMENDA` e modo "para estoque".
- `MotivoDescarte`: enum de motivos de descarte.
- `StatusPrecoPraticado`: `ABAIXO_DO_IDEAL`, `DENTRO_DO_IDEAL`, `ACIMA_DO_IDEAL`.

## Observações finais do backend

1. **Recálculo automático de precificação** é a lógica mais interconectada:
   mudanças em custo de ingrediente, custo/hora de recurso, ou taxas de canal
   disparam eventos que recalculam preços de produtos afetados silenciosamente.
2. **Entrega de pedido** (`PATCH /pedidos/{id}/status-pedido` → `ENTREGUE`) é
   a rota com mais efeitos colaterais: mexe em estoque de ingredientes,
   embalagem, produtos prontos e cria lançamento de caixa, tudo em uma
   transação, com validação prévia completa.
3. **Nenhuma rota tem paginação real** — listagens usam `findAll()` + filtros
   em memória.
4. **Segurança real está fora do código Java** (API Gateway/Lambda Authorizer
   da AWS); não há controle de papel no backend.
