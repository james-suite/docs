---
id: enum-calendar-event-state
title: Enum — CalendarEventState
description: Valores e invariantes propostos de CalendarEventState.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, agenda-tarefas
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Scheduled` | `scheduled` | Compromisso planejado |
| `Completed` | `completed` | Marcado como realizado |
| `Cancelled` | `cancelled` | Cancelado; continua no histórico da agenda |

## Regras

scheduled → completed/cancelled. Reabertura explícita preserva histórico. Exclusão é uma operação distinta que move o evento para lixeira com `deleted_at`; não representar exclusão como cancelamento. Passar o horário do compromisso não prova realização; não marcar como realizado automaticamente.
