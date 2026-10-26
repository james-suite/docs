---
id: enum-notification-channel
title: Enum — NotificationChannel
description: Valores e invariantes propostos de NotificationChannel.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, notificacoes
source_refs:
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `InApp` | `in_app` | Central interna |
| `Email` | `email` | E-mail |
| `Telegram` | `telegram` | Telegram |

## Regras e transições

Canal externo tem histórico de entregas independente da leitura interna. Push exige nova capacidade e decisão; não habilitar só porque um nome existe em enum.

## Persistência

Aplicar em `notification_deliveries` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:notificacoes) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
