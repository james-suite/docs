---
id: contribuicao
title: Contribuição
description: Caminho recomendado para preparar o ambiente, entender as convenções e validar mudanças no James.
type: guide
status: observed
visibility: public
tags: contribuição, desenvolvimento, testes
related: primeiros-passos, arquitetura, decisoes, traits-e-helpers
source_refs: https://github.com/james-suite/james/blob/master/README.md, https://github.com/james-suite/james/blob/master/composer.json, https://github.com/james-suite/james/blob/master/phpunit.xml
---

## Antes de alterar o código

Leia [Primeiros passos](doc:primeiros-passos) para preparar o ambiente e [Stack e tecnologias](doc:tecnologias) para conhecer as dependências principais. As [decisões arquiteturais](doc:decisoes) registram escolhas que devem ser consideradas antes de introduzir uma nova abstração.

## Ciclo local

1. Faça a mudança no repositório da aplicação.
2. Execute os testes Pest relacionados e, quando necessário, a suíte completa.
3. Formate o PHP com Laravel Pint.
4. Verifique o comportamento da interface com o servidor e o Vite quando a mudança for visual.

Os comandos concretos estão no guia de instalação; esta página não substitui as instruções operacionais do repositório.

## Escopo do projeto

O James é um projeto pessoal e auto-hospedado. A documentação pública descreve o estado observado do código e não promete compatibilidade de uma API pública ou suporte multiusuário.

## Onde continuar

- [Traits e helpers](doc:traits-e-helpers) para padrões reutilizados no código;
- [Roadmap](doc:roadmap) para módulos planejados;
- [Repositório da aplicação](https://github.com/james-suite/james) para abrir uma mudança;
- [Repositório da documentação](https://github.com/james-suite/docs) para corrigir ou ampliar estas páginas.
