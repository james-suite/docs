---
id: primeiros-passos
title: Primeiros passos
description: Instalação do ambiente de desenvolvimento local do James com Laravel Sail.
type: guide
status: observed
visibility: public
tags: instalação, desenvolvimento, sail
related: home, autenticacao, contribuicao, deploy
source_refs: https://github.com/james-suite/james/blob/master/compose.yaml, https://github.com/james-suite/james/blob/master/composer.json, https://github.com/james-suite/james/blob/master/package.json
---

## Pré-requisitos

Tenha [Docker](https://docs.docker.com/engine/install/) e Docker Compose, [Composer](https://getcomposer.org/) e [Node.js com npm](https://nodejs.org/) instalados. O ambiente local usa Laravel Sail para executar PHP, PostgreSQL e Mailpit.

## Instalação

1. Clone o [repositório do James](https://github.com/james-suite/james) e entre na pasta do projeto.
2. Instale as dependências PHP com `composer install`.
3. Copie `.env.example` para `.env`.
4. Gere a chave da aplicação com `./vendor/bin/sail artisan key:generate`.
5. Inicie os containers com `./vendor/bin/sail up -d`.
6. Crie o link de storage com `./vendor/bin/sail artisan storage:link`.
7. Execute as migrações e seeds com `./vendor/bin/sail artisan migrate --seed`.
8. Instale as dependências do front-end com `./vendor/bin/sail npm install`.
9. Inicie o Vite com `./vendor/bin/sail npm run dev` ou gere os assets com `npm run build`.

### Variáveis de ambiente

O `.env.example` já aponta o banco para o serviço Docker `pgsql` e o e-mail local para o Mailpit. Confira especialmente `APP_URL=http://localhost`, `MEDIA_DISK=private`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` e `NOTIFICATIONS_MAIL_ENABLED`. Os valores de Telegram ficam vazios por padrão no desenvolvimento.

## Testes e rotina local

O James usa Pest para testes de unidade e feature. Na primeira execução, crie o banco de testes quando necessário; depois use `./vendor/bin/sail test` ou `./vendor/bin/sail artisan test --compact`. Para uma execução direcionada, acrescente o nome do arquivo ou `--filter=Notification`.

Durante os testes, `phpunit.xml` desativa os canais externos de Telegram e e-mail.

O scheduler pode ser mantido em execução com `./vendor/bin/sail artisan schedule:work`; a fila pode ser atendida com `./vendor/bin/sail artisan queue:listen --tries=1 --timeout=0`. Os comandos financeiros também podem ser executados manualmente:

```bash
./vendor/bin/sail artisan finance:process-recurrences
./vendor/bin/sail artisan finance:rollover-invoices
./vendor/bin/sail artisan finance:rollover-transactions
```

## Serviços locais

- A aplicação é exposta pela porta configurada em `APP_PORT`.
- O PostgreSQL usa `FORWARD_DB_PORT` quando essa variável é definida.
- A aplicação fica disponível em [http://localhost](http://localhost).
- O Mailpit oferece SMTP e painel web em [http://localhost:8025](http://localhost:8025).
- O Vite usa a porta `5173` por padrão.

## Comandos úteis

Um alias `sail` pode simplificar os comandos repetidos. No shell, ele pode apontar para `vendor/bin/sail`. Os comandos mais usados são `sail down`, `sail shell`, `sail bin pint` e `sail artisan pail`.

Consulte [Autenticação](doc:autenticacao) depois do primeiro acesso e [Deploy](doc:deploy) para o ambiente de produção.
