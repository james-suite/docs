---
id: visao-geral
title: Visão geral
description: Origem, objetivos e escolhas que definem o James como um Life OS pessoal.
type: overview
status: observed
visibility: public
tags: produto, contexto, stack
related: home, funcionalidades, tecnologias, roadmap
source_refs: https://github.com/james-suite/james/blob/master/README.md, https://github.com/james-suite/james/blob/master/composer.json, https://github.com/james-suite/james/blob/master/package.json
---

## O que é o James

O James é um ERP pessoal e auto-hospedado. A proposta é manter uma fonte única de verdade para informações que normalmente ficam espalhadas entre planilhas e aplicações: contatos, finanças e acertos de despesas.

O projeto segue uma filosofia opinativa (*Omakase*): as funcionalidades são priorizadas de acordo com o fluxo de trabalho do próprio projeto. Os módulos futuros listados no [roadmap](doc:roadmap) são planos, não funcionalidades já disponíveis.

## Escolhas técnicas

- **Back-end:** PHP 8.5 e Laravel 13.
- **Autenticação:** Laravel Fortify com views Blade próprias.
- **Interface:** Blade, Alpine.js e Tailwind CSS v4.
- **Banco de dados:** PostgreSQL, incluindo `unaccent` e `pg_trgm` para buscas.
- **Visualizações:** Apache ECharts integrado por JavaScript.
- **Arquivos privados:** Spatie Media Library em discos privados para avatares e comprovantes.

Os detalhes e as referências de implementação estão em [Stack e tecnologias](doc:tecnologias) e no [mapa de arquitetura](doc:arquitetura).

## Como ler esta documentação

As páginas distinguem funcionalidades ativas, decisões registradas e planos futuros. O conteúdo técnico aponta para arquivos do repositório da aplicação sempre que uma afirmação depende da implementação.
