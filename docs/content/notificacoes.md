---
id: notificacoes
title: Notificações
description: Alertas persistidos no banco, filtráveis no painel e distribuídos opcionalmente por Telegram e e-mail.
type: feature
status: observed
visibility: public
tags: notificações, telegram, e-mail, filas
related: automacoes, financas, nfce, auditoria, dashboard
source_refs: https://github.com/james-suite/james/blob/master/routes/web.php, https://github.com/james-suite/james/blob/master/app/Http/Controllers/NotificationController.php, https://github.com/james-suite/james/blob/master/app/Notifications/GeneralNotification.php, https://github.com/james-suite/james/blob/master/app/Notifications/DueTodayNotification.php, https://github.com/james-suite/james/blob/master/app/Notifications/FinancialSummaryNotification.php, https://github.com/james-suite/james/blob/master/app/Console/Commands/SendDueTodayAlerts.php, https://github.com/james-suite/james/blob/master/app/Console/Commands/SendMonthlyFinancialDigest.php, https://github.com/james-suite/james/blob/master/app/Jobs/ScrapeNfceInvoiceJob.php
---

## O que o módulo entrega

Notificações é a camada de comunicação do James. Ela transforma eventos de finanças, automações, importações e rotinas em mensagens com título, nível, detalhes e uma ação que leva de volta ao sistema.

Os fluxos automáticos persistem suas notificações no banco de dados. `GeneralNotification` faz isso quando `database` está presente em `channels`; Telegram e e-mail são canais complementares e podem estar desabilitados sem impedir o registro interno quando ele foi solicitado.

{{diagram:notifications-flow}}

## Contrato de uma notificação

`GeneralNotification` recebe um payload estável:

```php
new GeneralNotification(
    title: 'Título curto',
    message: 'Explicação do que aconteceu.',
    actionUrl: route('financial.dashboard'),
    level: NotificationLevel::Warning,
    details: ['Valor' => 'R$ 150,00'],
    channels: ['database', 'telegram', 'mail'],
    actionLabel: 'Abrir painel',
    items: [],
)
```

| Campo | Uso |
| --- | --- |
| `title` | Título exibido no painel, e-mail e Telegram. |
| `message` | Texto principal da ocorrência. |
| `action_url` | Link opcional para a tela relacionada. |
| `action_label` | Texto do botão da ação. |
| `level` | `info`, `success`, `warning` ou `danger`. |
| `details` | Mapa chave/valor para metadados legíveis. |
| `items` | Lista de itens com descrição, quantidade, preço unitário e total. |
| `channels` | Canais desejados para `GeneralNotification`. |

As notificações específicas de vencimentos e resumo financeiro usam payloads adicionais (`due_alert` e `financial_summary`) para renderizar blocos estruturados na interface e nas mensagens externas.

## Níveis

| Case | Valor | Uso visual |
| --- | --- | --- |
| `NotificationLevel::Info` | `info` | Informação ou conclusão sem alerta. |
| `NotificationLevel::Success` | `success` | Operação concluída com êxito. |
| `NotificationLevel::Warning` | `warning` | Prazo, pendência ou atenção necessária. |
| `NotificationLevel::Danger` | `danger` | Erro, falha ou risco financeiro. |

Cada nível fornece rótulo, cor e ícone Heroicon. No e-mail, o nível `Danger` usa o estilo de erro do Laravel Mail; no Telegram, o nível aparece em caixa alta no cabeçalho.

## De onde as notificações vêm

### Alertas de vencimentos

O comando `finance:due-today-alerts` procura itens para hoje e amanhã em três fontes:

- transações pendentes ou efetivadas que se enquadram no período;
- faturas de cartão não pagas com vencimento no período;
- recorrências ativas ainda não materializadas, sem duplicar as recorrências já representadas pela fatura do cartão.

O alerta consolida quantidade, receitas, despesas, impacto líquido e a lista de dias/itens. Ele não é enviado quando não há itens. Um cache por usuário impede reenvio do mesmo dia; `--force` permite reenviar manualmente.

### Resumo financeiro mensal

O comando `finance:monthly-digest` calcula o mês anterior e compara receitas, despesas e resultado com o mês anterior a ele. O resumo inclui:

- receitas, despesas e resultado do período;
- variações em relação ao período comparado;
- saldo atual das contas;
- compromissos pendentes;
- saldo líquido;
- distribuição por categorias de receita e despesa.

O envio mensal também usa uma chave de cache por usuário e período. `--force` permite repetir o resumo quando necessário.

### Rotinas e importação de NFC-e

O processamento de recorrências, a rolagem de faturas e outras automações usam `GeneralNotification` para informar sucesso ou falha. A importação assíncrona de NFC-e envia uma notificação com ação para abrir o rascunho ou tentar novamente. Consulte [Rotinas automáticas](doc:automacoes) e [Importação de NFC-e](doc:nfce).

## Canais de entrega

### Banco de dados

É o canal interno e alimenta `/notifications`. O JSON persistido contém o payload da notificação e permite renderizar detalhes, itens, nível e ação sem depender do canal externo.

### Telegram

O canal é usado somente quando `TELEGRAM_BOT_TOKEN` e `TELEGRAM_CHAT_ID` estão preenchidos. A mensagem inclui título, texto, detalhes, itens e botão de ação quando a URL é externa.

Se a URL aponta para `localhost` ou `127.0.0.1`, o James não cria um botão inline inválido para a API do Telegram; ele coloca o endereço como texto seguro na mensagem.

### E-mail

O e-mail exige destinatário com endereço preenchido e `NOTIFICATIONS_MAIL_ENABLED=true`. A mensagem usa os templates transacionais do Laravel, inclui detalhes e itens e adiciona um botão quando existe `actionUrl`.

### Filas

`GeneralNotification`, `DueTodayNotification` e `FinancialSummaryNotification` implementam `ShouldQueue`. Em produção, o worker precisa estar ativo para que as notificações queued sejam processadas. O registro no banco, o envio externo e a disponibilidade da fila devem ser tratados como partes distintas do fluxo.

## Central `/notifications`

O painel autenticado permite:

- pesquisar no payload JSON da notificação;
- filtrar por `unread` ou `read`;
- filtrar por data inicial e final;
- ordenar do mais novo para o mais antigo ou vice-versa;
- navegar por páginas de 20 registros;
- abrir uma notificação, marcando-a como lida;
- marcar todas as notificações como lidas;
- excluir uma notificação individual.

O contador da sidebar e o cartão do Dashboard usam somente notificações não lidas. A tela de detalhes verifica se a notificação pertence ao usuário autenticado antes de exibi-la ou alterá-la.

## Exemplos para desenvolvimento

### Notificação somente interna

```php
use App\Notifications\GeneralNotification;

$user->notify(new GeneralNotification(
    title: 'Rascunho pronto',
    message: 'A NFC-e foi importada e aguarda revisão.',
    actionUrl: route('financial.transactions.edit', $transaction),
    channels: ['database'],
));
```

### Notificação com itens

```php
$user->notify(new GeneralNotification(
    title: 'Importação concluída',
    message: 'Revise os itens antes de efetivar a transação.',
    details: ['Emitente' => 'Comércio exemplo', 'Total' => 'R$ 120,00'],
    items: [
        ['description' => 'Produto', 'quantity' => '2', 'unit_price' => 'R$ 60,00', 'total' => 'R$ 120,00'],
    ],
));
```

Nos testes, use `Notification::fake()` e faça asserções por destinatário, classe, nível, título e payload. O projeto mantém testes de unidade para os três formatos de notificação e testes de feature para a central web.

### Referências

- [Rotinas automáticas](doc:automacoes) — comandos que produzem alertas.
- [Finanças](doc:financas) — origem de vencimentos e resumos.
- [Auditoria e logs](doc:auditoria) — rastreia as mutações que deram origem a parte dos avisos.
