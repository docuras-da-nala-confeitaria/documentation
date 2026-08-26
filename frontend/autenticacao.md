---
layout: default
title: Login / Autenticação
parent: Frontend
nav_order: 1
---

# Login / Autenticação

- **`/login`**: e-mail + senha, `required` HTML; toggle mostrar/ocultar
  senha; 401 → "E-mail ou senha inválidos."; sucesso navega para
  `/dashboard`.
- **`/esqueci-senha`**: **sempre mostra mensagem de sucesso**,
  independente do resultado real (proteção contra enumeração de e-mails —
  mesmo padrão do backend).
- **`/redefinir-senha?token=`**: valida senha ≥ 6 caracteres, confirmação
  igual, token presente na URL; erro de API sempre vira mensagem genérica
  "Este link expirou ou é inválido."
- **Layout `(app)`**: guarda de rota — enquanto carrega sessão mostra
  "Carregando..."; sem usuário, `router.replace("/login")` e renderiza
  `null` (evita flash de conteúdo protegido).
- **Sidebar/Header**: navegação fixa sem filtro por papel; botão "Sair"
  chama `logout()`.
