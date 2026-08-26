---
layout: default
title: Infraestrutura Raspberry Pi
parent: Arquitetura
nav_order: 2
---

# Infraestrutura Raspberry Pi (local / on-premises)

Ambiente alternativo ao AWS, pensado para rodar o sistema numa rede local
(casa ou loja), sem depender de internet nem gerar custo de nuvem — útil
como ambiente de uso real de baixo custo ou como ambiente de testes antes
de ir para produção na AWS.

## Como funciona

Toda a stack sobe via **Docker Compose** em um único host (mini PC ou
Raspberry Pi) na rede local:

- Um container **PostgreSQL**, com volume nomeado para persistir os
  dados entre reinícios.
- Um container do **backend** Spring Boot, buildado a partir do
  Dockerfile do próprio repositório do backend.
- Um container do **frontend** Next.js, também buildado localmente,
  apontando para o IP do backend na rede local.

Qualquer dispositivo na mesma rede (celular, outro computador) acessa o
sistema pelo IP fixo reservado do host, sem precisar de VPN ou exposição à
internet.

## Backup

Uma rotina agendada (cron) faz `pg_dump` do banco periodicamente,
compacta o resultado, mantém uma janela de retenção local e opcionalmente
envia uma cópia para armazenamento em nuvem (OneDrive, via `rclone`) como
segunda camada de proteção contra perda do host físico.

## Migração entre ambientes

Existe um processo documentado para mover os dados de um ambiente de
teste (ex.: um PC) para a Raspberry Pi definitiva quando ela estiver
disponível: gerar um dump do banco, transferir o arquivo (pendrive, rede
local ou nuvem), subir a stack vazia na Pi e restaurar o dump antes de
qualquer uso real, para evitar conflitos de dados.

## O que fica fora desta página

Comandos exatos, nomes de scripts e variáveis de ambiente específicas
vivem em `sistemas/infra-rasp/` (`BACKUP.md`, `MIGRACAO.md`,
`docker-compose.local.yml`, `setup-raspberry-pi.sh`) no repositório do
sistema — este ambiente roda inteiramente dentro de uma rede privada, sem
exposição pública, então o detalhamento operacional não precisa estar num
site público.
