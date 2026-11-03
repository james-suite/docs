---
id: enum-notification-channel
title: Enum — NotificationChannel
description: Valores e invariantes propostos de NotificationChannel.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, notificacoes
source_refs: https://developer.apple.com/documentation/usernotifications/sending-web-push-notifications-in-web-apps-and-browsers, https://www.rfc-editor.org/rfc/rfc8030, https://www.rfc-editor.org/rfc/rfc8292
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `InApp` | `in_app` | Central interna |
| `Email` | `email` | E-mail |
| `WebPush` | `web_push` | Notificação push do navegador/web app instalado |

## Regras e transições

Canal externo tem histórico de entrega independente da leitura interna. `WebPush` só fica disponível quando o navegador oferece suporte e a pessoa autoriza uma assinatura para aquela instalação. O usuário pode revogar a assinatura; falha ou expiração não apaga a notificação interna.
