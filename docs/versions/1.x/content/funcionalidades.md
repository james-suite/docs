---
id: funcionalidades
title: Funcionalidades
description: Mapa dos módulos ativos, das integrações entre eles e dos recursos ainda planejados.
type: overview
status: observed
visibility: public
tags: módulos, produto, navegação, integrações
related: home, dashboard, contatos, financas, acertos, notificacoes, auditoria, automacoes, roadmap
source_refs: https://github.com/james-suite/james/blob/master/README.md, https://github.com/james-suite/james/blob/master/routes/web.php, https://github.com/james-suite/james/blob/master/routes/contacts.php, https://github.com/james-suite/james/blob/master/routes/financial.php, https://github.com/james-suite/james/blob/master/routes/settlements.php, https://github.com/james-suite/james/blob/master/routes/console.php
---

## Como o produto é organizado

O James separa o cadastro pessoal, a competência de acertos e o caixa financeiro em módulos conectados. Essa separação evita que uma dívida informal seja confundida com uma movimentação bancária e permite que cada tela tenha uma regra clara.

| Área | Responsabilidade | Entrada principal |
| --- | --- | --- |
| [Dashboard](doc:dashboard) | Triagem diária e indicadores rápidos. | `/dashboard` |
| [Contatos](doc:contatos) | Pessoas, grupos, notas e avatares. | `/contacts` |
| [Finanças](doc:financas) | Contas, transações, cartões, faturas, recorrências, tags e relatórios. | `/financial` |
| [Acertos](doc:acertos) | Saldos pessoais e despesas compartilhadas. | `/settlements` |
| [Notificações](doc:notificacoes) | Alertas internos e canais externos opcionais. | `/notifications` |
| [Auditoria e logs](doc:auditoria) | Histórico das mutações de dados de negócio. | `/audit` |
| [Rotinas automáticas](doc:automacoes) | Scheduler, filas e materialização de dados. | comandos Artisan |

## Integrações entre módulos

### Contatos → Acertos

Um acerto sempre aponta para um contato. O módulo de Contatos é a fonte do nome, avatar, categoria e grupos usados nas telas de saldos e rateios.

### Acertos → Finanças

Um lançamento ou uma divisão pode criar uma transação financeira opcional. A relação por `financial_transaction_id` permite navegar entre a promessa de pagamento e a entrada/saída efetiva do caixa.

### Finanças → Notificações

Scheduler, vencimentos de faturas, recorrências, resumos mensais e importações de NFC-e usam a central de notificações para entregar contexto e uma ação de retorno.

### Todos os módulos → Auditoria

Models de negócio que usam `LogsActivity` deixam um diff dos campos alterados. Operações automáticas podem aparecer como realizadas pelo sistema quando não existe usuário autenticado.

## Status da documentação

As páginas marcadas como `observed` descrevem comportamentos encontrados no código atual. Itens do [Roadmap](doc:roadmap) são intenções futuras e não devem ser tratados como funcionalidades disponíveis.

Os módulos planejados incluem veículos e saúde/treinos. Eles permanecem separados do mapa de funcionalidades ativas para evitar expectativa de telas que ainda não existem.

## Por onde começar

- Usuário novo: [Primeiros passos](doc:primeiros-passos), [Dashboard](doc:dashboard) e [Visão geral](doc:visao-geral).
- Uso financeiro: [Finanças](doc:financas), [Painel financeiro](doc:painel-financeiro) e [Relatórios](doc:relatorios).
- Organização pessoal: [Contatos](doc:contatos) e [Acertos](doc:acertos).
- Desenvolvimento: [Arquitetura](doc:arquitetura), [Rotinas automáticas](doc:automacoes) e [Contribuição](doc:contribuicao).
