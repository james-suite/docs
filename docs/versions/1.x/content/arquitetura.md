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

## Navegação e experiência de interface

- **Preservação de filtros e paginação:** o middleware `RememberListingUrl` grava em sessão a última URL de listagem (rotas `.index` e `.history`) por caminho. Quando o usuário abre um registro, edita ou navega por páginas intermediárias e clica em **Voltar**, o `SafeBackUrl::resolveFallback()` recupera automaticamente os filtros e a página anterior da listagem.
- **Atalhos e leitura de dados:** a barra de filtros (`x-filter-bar`) suporta atalho `/` no teclado para focar a busca e botão `×` interno para limpar o termo; as abas do navegador exibem títulos dinâmicos por módulo (`<Módulo> — James`); toasts pausam o temporizador ao passar o mouse (`4s`, ou `5s` para erro); e valores monetários utilizam `tabular-nums` para alinhamento vertical.

## Fontes de decisão

As escolhas e seus motivos estão registradas em [Decisões arquiteturais](doc:decisoes). Para a visão de tecnologias, consulte [Stack e tecnologias](doc:tecnologias).
