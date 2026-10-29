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
updated: 2026-10-28
---

## Contrato proposto

Enum string planejado, ainda sem classe PHP no 2.0. Cases em TitleCase e valores persistidos estáveis.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Scheduled` | `scheduled` | Compromisso planejado |
| `Completed` | `completed` | Marcado como realizado |
| `Cancelled` | `cancelled` | Cancelado; continua no histórico da agenda |

## Regras

scheduled → completed/cancelled. Reabertura explícita preserva histórico. Exclusão é uma operação distinta que move o evento para lixeira com `deleted_at`; não representar exclusão como cancelamento. Passar o horário do compromisso não prova realização; não marcar como realizado automaticamente.

## Persistência e implementação

Usar em `calendar_events` com cast Eloquent e validação/constraints. Aplicar no fluxo de [domínio](doc:agenda-tarefas); a presença de um case não habilita automaticamente uma funcionalidade.

- [ ] Confirmar regras e transições com exemplos.
- [ ] Criar enum, rótulos e cast na implementação de 2027.
- [ ] Validar valores inválidos, escopo e conversão de datas/valores aplicável.
