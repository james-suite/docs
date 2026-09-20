---
id: automacoes
title: Rotinas automáticas
description: Scheduler, comandos financeiros, filas e notificações executados fora da requisição web.
type: architecture
status: observed
visibility: public
tags: automação, scheduler, filas, finanças, notificações
related: financas, painel-financeiro, notificacoes, auditoria, deploy
source_refs: https://github.com/james-suite/james/blob/master/routes/console.php, https://github.com/james-suite/james/tree/master/app/Console/Commands, https://github.com/james-suite/james/blob/master/app/Jobs/ScrapeNfceInvoiceJob.php, https://github.com/james-suite/james/blob/master/config/queue.php
---

## Dois mecanismos diferentes

O James usa dois processos complementares:

- **Scheduler** decide quando comandos devem rodar.
- **Fila** processa jobs e notificações que não precisam bloquear a requisição web.

Em desenvolvimento, o scheduler pode ser executado com `php artisan schedule:work` e a fila com `php artisan queue:listen`. Em produção, ambos precisam de processos supervisionados; consulte [Deploy](doc:deploy) para o ambiente da aplicação.

## Agenda registrada em `routes/console.php`

| Frequência | Comando | Responsabilidade |
| --- | --- | --- |
| Diária | `finance:rollover-invoices` | Garante as faturas de cartão dos períodos necessários. |
| Diária | `finance:rollover-transactions` | Atualiza transações pendentes vencidas para continuarem visíveis. |
| Diária | `finance:process-recurrences` | Materializa recorrências cuja próxima data chegou. |
| Diária às 08:00 | `finance:due-today-alerts` | Envia vencimentos de hoje e amanhã. |
| Dia 1 de cada mês às 09:00 | `finance:monthly-digest` | Envia o resumo financeiro do mês anterior. |

`process-recurrences`, `due-today-alerts` e `monthly-digest` usam `withoutOverlapping()` e `onOneServer()` para evitar concorrência entre executores. O scheduler sozinho não cria os processos: ele precisa permanecer em execução para avaliar os horários.

## Processamento das rotinas

### Recorrências

`finance:process-recurrences` procura recorrências ativas cuja `next_processing_date` chegou ou passou. A transação materializada recebe o tipo, valor, conta ou cartão e tags da recorrência; depois, a próxima data é avançada de acordo com a frequência semanal, mensal ou anual.

Para cartões, a compra é vinculada à fatura correspondente e permanece pendente até o fluxo de pagamento da fatura. Para contas, a transação pode ser registrada como efetivada conforme a regra do comando.

### Faturas de cartão

`finance:rollover-invoices` garante que exista a fatura referente ao período atual de cada cartão ativo. As datas de fechamento e vencimento são calculadas pelas regras do cartão, incluindo ajustes de dias úteis.

### Transações pendentes

`finance:rollover-transactions` evita que uma transação pendente vencida desapareça de uma projeção antiga. O comando desloca a data para o dia atual para que a pendência continue visível até ser paga, editada ou removida.

### Vencimentos próximos

`finance:due-today-alerts` consolida transações, faturas e recorrências de hoje e amanhã. O comando não envia mensagem quando não encontra itens e usa cache por usuário/data para não duplicar alertas. O uso manual pode forçar um novo envio:

```bash
php artisan finance:due-today-alerts --force
```

Veja os detalhes do payload e dos canais em [Notificações](doc:notificacoes).

### Resumo mensal

`finance:monthly-digest` compara o mês anterior com o mês anterior a ele e envia receitas, despesas, resultado, variações, saldo das contas, compromissos pendentes e categorias. Também usa cache por usuário/período e aceita:

```bash
php artisan finance:monthly-digest --force
```

## Fila e jobs

A importação de NFC-e despacha `ScrapeNfceInvoiceJob`, que consulta o provedor fiscal, cria o rascunho e notifica o usuário. Notificações como `GeneralNotification`, `DueTodayNotification` e `FinancialSummaryNotification` também são queued.

Sem um worker, o scheduler ainda pode executar os comandos, mas as entregas assíncronas ficarão aguardando na fila. Em ambiente local:

```bash
./vendor/bin/sail artisan schedule:work
./vendor/bin/sail artisan queue:listen --tries=1 --timeout=0
```

## Auditoria e segurança operacional

As mutações executadas pelas rotinas passam pelos mesmos Models e serviços das operações web. Por isso, podem aparecer no [módulo de Auditoria](doc:auditoria). Como comandos e jobs normalmente não possuem sessão autenticada, o autor pode ser exibido como **Sistema / Rotina Automática**.

Os comandos de notificação usam chaves de cache para limitar reenvios. Em caso de exceção, a chave é removida para permitir nova tentativa em vez de marcar uma execução falha como concluída.

## Executar comandos manualmente

Para investigar uma instalação local, os comandos podem ser executados individualmente:

```bash
./vendor/bin/sail artisan finance:rollover-invoices
./vendor/bin/sail artisan finance:rollover-transactions
./vendor/bin/sail artisan finance:process-recurrences
./vendor/bin/sail artisan finance:due-today-alerts
./vendor/bin/sail artisan finance:monthly-digest
```

Verifique a saída do Artisan, o painel financeiro, a central de notificações e o activity log depois da execução. Isso torna mais fácil separar uma falha no cálculo, uma falha na persistência e uma falha de entrega externa.
