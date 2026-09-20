---
id: dashboard
title: Dashboard
description: Visão diária do saldo financeiro, acertos pendentes, contatos cadastrados e notificações não lidas.
type: feature
status: observed
visibility: public
tags: dashboard, produto, indicadores
related: funcionalidades, financas, acertos, contatos, notificacoes
source_refs: https://github.com/james-suite/james/blob/master/routes/web.php, https://github.com/james-suite/james/blob/master/app/Http/Controllers/DashboardController.php, https://github.com/james-suite/james/blob/master/resources/views/dashboard.blade.php, https://github.com/james-suite/james/blob/master/app/Services/FinanceDashboardService.php
---

## Para que serve

O Dashboard (`/dashboard`) é a primeira visão depois da autenticação. Ele não substitui os módulos detalhados; funciona como um painel de triagem para decidir onde agir hoje.

Os números são calculados na data atual e respeitam os registros ativos do usuário. Contatos com acertos arquivados não entram no resumo principal de acertos.

## Indicadores principais

### Finanças

Mostra o saldo atual das contas financeiras não classificadas como investimento e leva para o [Painel financeiro](doc:painel-financeiro). Esse cartão representa o saldo das contas, não o saldo líquido depois de faturas e compromissos pendentes.

O painel financeiro detalhado calcula também o saldo líquido, descontando faturas abertas e despesas pendentes.

### Acertos

Mostra o valor líquido agregado dos contatos com pendências e informa se o usuário tem a receber, tem a pagar ou está quite. O cálculo usa o mesmo `SettlementBalanceCalculator` da página de [Acertos](doc:acertos).

### Contatos

Exibe a quantidade de contatos cadastrados e abre a listagem de [Contatos](doc:contatos). O contador não é uma soma de pessoas únicas em grupos; ele representa os registros da tabela de contatos.

### Notificações

Exibe a quantidade de notificações não lidas e abre a central de [Notificações](doc:notificacoes).

## Blocos de atenção

Quando existem dados, o Dashboard apresenta dois blocos adicionais:

- **Acertos pendentes**: até quatro contatos com saldo diferente de zero, ordenados pelo maior valor absoluto. Cada item abre o ledger do contato.
- **Notificações não lidas**: até três notificações mais recentes, com título, mensagem e data. Abrir uma delas leva ao detalhe e marca a notificação como lida.

Se não houver acertos pendentes nem notificações não lidas, esses blocos não ocupam espaço na página.

## O que o Dashboard não faz

- não edita transações, contatos ou acertos;
- não apresenta o histórico financeiro completo;
- não substitui os gráficos e projeções do Painel financeiro;
- não inclui contatos arquivados no resumo de acertos;
- não transforma notificações em tarefas ou mensagens enviadas automaticamente.

Use o Dashboard para identificar uma pendência e siga o link para o módulo responsável pela operação.
