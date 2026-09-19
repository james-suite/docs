---
id: arquitetura
title: Arquitetura
description: Visão de alto nível dos limites da aplicação James e de seus serviços de suporte.
type: architecture
status: observed
visibility: public
tags: arquitetura, laravel, postgres, filas
related: visao-geral, tecnologias, autenticacao, automacoes, contribuicao
source_refs: https://github.com/james-suite/james/blob/master/compose.yaml, https://github.com/james-suite/james/blob/master/routes/web.php, https://github.com/james-suite/james/blob/master/config/queue.php, https://github.com/james-suite/james/blob/master/config/filesystems.php
diagram: james-overview
---

## Limites principais

O James é uma aplicação Laravel renderizada no servidor. O navegador acessa rotas protegidas, o domínio da aplicação coordena regras de negócio e persistência, o PostgreSQL guarda os dados relacionais e os arquivos sensíveis ficam em discos privados.

{{diagram:james-overview}}

Em desenvolvimento, o Laravel Sail orquestra a aplicação, o PostgreSQL e o Mailpit. A fila padrão usa o banco de dados; rotinas financeiras são registradas no scheduler do Laravel e podem ser executadas continuamente em produção.

## Fontes de decisão

As escolhas e seus motivos estão registradas em [Decisões arquiteturais](doc:decisoes). Para a visão de tecnologias, consulte [Stack e tecnologias](doc:tecnologias).
