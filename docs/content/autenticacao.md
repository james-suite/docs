---
id: autenticacao
title: Autenticação
description: Como o James apresenta login, recuperação de senha e proteção das áreas autenticadas.
type: feature
status: observed
visibility: public
tags: autenticação, segurança, fortify
related: primeiros-passos, arquitetura, auditoria
source_refs: https://github.com/james-suite/james/blob/master/app/Providers/FortifyServiceProvider.php, https://github.com/james-suite/james/blob/master/config/fortify.php, https://github.com/james-suite/james/blob/master/routes/web.php
---

## Visão geral

O James usa o Laravel Fortify como backend de autenticação e renderiza as telas com views Blade próprias. A raiz da aplicação redireciona para `/login`; as áreas de negócio ficam dentro de um grupo protegido pelo middleware `auth`.

## Fluxos disponíveis

- login por e-mail e senha;
- solicitação e redefinição de senha;
- atualização do perfil;
- atualização da senha;
- redirecionamento para `/dashboard` depois da autenticação.

O username configurado para o Fortify é `email`, e o limiter de login permite cinco tentativas por minuto por combinação de e-mail e endereço IP.

## Limites documentados

Passkeys e autenticação de dois fatores aparecem apenas como opções comentadas na configuração atual. Não são descritas aqui como funcionalidades ativas.
