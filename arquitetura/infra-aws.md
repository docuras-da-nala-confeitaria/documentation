---
layout: default
title: Infraestrutura AWS
parent: Arquitetura
nav_order: 1
---

# Infraestrutura AWS

Infraestrutura como código via Terraform, com dois ambientes independentes
(`dev` e `prod`), cada um com seu próprio state. Os dois ambientes
compartilham os mesmos módulos, variando apenas os valores de entrada
(ex.: `dev` sem proteção contra deleção e com desligamento automático fora
do horário comercial; `prod` sempre ligado, com banco em multi-AZ).

## Componentes principais

- **API Gateway** — único ponto de entrada público da API. Todo tráfego é
  HTTPS por padrão do próprio serviço gerenciado.
- **Lambda Authorizer** — valida a assinatura e expiração do JWT antes de
  qualquer requisição alcançar o backend. Sem cache de resultado (TTL 0).
  A chave de assinatura é compartilhada com o backend via um secret
  dedicado.
- **ALB interno** — sem IP público; só aceita tráfego vindo do API
  Gateway. Encaminha para o backend.
- **ECS Fargate** — roda o backend Spring Boot em containers, numa subnet
  privada sem rota direta para a internet (acesso a serviços AWS via VPC
  Endpoints).
- **RDS PostgreSQL** — banco de dados, em subnet isolada. Credenciais
  nunca aparecem em texto plano: geradas e injetadas via Secrets Manager
  diretamente nas variáveis de ambiente do container, no momento do
  start.
- **S3 + CloudFront** — hospedam o build estático do frontend Next.js.
  Bucket privado, acesso só via Origin Access Control do CloudFront.
- **SNS** — canal de evento para o fluxo de "esqueci minha senha". Hoje
  funciona como notificação/auditoria para um e-mail fixo cadastrado, não
  como entrega real ao e-mail dinâmico de quem solicitou a redefinição
  (isso exigiria integração futura com um serviço de envio transacional).
- **Lambda do cardápio + ALB interno próprio** — gera a imagem do
  cardápio (Pillow), lendo os produtos direto do RDS. Empacotada como
  imagem de container (dependência nativa do Pillow não cabe num zip
  simples). Fica atrás do seu próprio ALB interno, exposto só ao
  backend (não ao API Gateway) — o backend continua chamando isso como
  uma URL HTTP comum, igual já funciona hoje em Docker; nenhuma
  mudança de código no backend é necessária pra essa migração.

## Rede

Três camadas de subnet: **pública** (hoje sem recursos residentes),
**privada** (containers do backend e o ALB interno) e **database** (RDS,
isolado). Não há NAT Gateway — o tráfego para serviços AWS a partir da
subnet privada passa por VPC Endpoints, não pela internet pública.

## O que fica fora desta página

Detalhes de módulos Terraform individuais, políticas IAM específicas,
nomes exatos de recursos e o script de desligamento automático do
ambiente `dev` não são reproduzidos aqui — são detalhes de implementação
sujeitos a mudança e vivem em `sistemas/infra-aws/` (README, `DEPLOY.md`,
`modules/`, `environments/`) no repositório do sistema.

Também fora do escopo desta infraestrutura, hoje: pipeline de CI/CD
(build e deploy automatizados — inclui a imagem da Lambda do cardápio,
cujo build/push ainda é manual), domínio próprio/certificado
customizado, e envio real de e-mail transacional.
