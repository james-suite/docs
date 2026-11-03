---
id: enum-notification-level
title: Enum — NotificationLevel
description: Valores e invariantes propostos de NotificationLevel.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, notificacoes
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Info` | `info` | Informação |
| `Success` | `success` | Conclusão útil |
| `Warning` | `warning` | Pendência ou prazo |
| `Danger` | `danger` | Falha ou atenção elevada |

## Regras e transições

Preserva valores atuais. Gravidade visual não define preferência de canal nem urgência por si só.
