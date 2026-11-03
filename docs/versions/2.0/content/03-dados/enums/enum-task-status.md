---
id: enum-task-status
title: Enum — TaskStatus
description: Valores e invariantes propostos de TaskStatus.
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
| `Todo` | `todo` | A fazer |
| `Doing` | `doing` | Em andamento |
| `Done` | `done` | Concluída |
| `Cancelled` | `cancelled` | Cancelada |

## Regras e transições

Transições e reabertura são auditadas. Concluir tarefa não efetiva pagamento nem cria manutenção; domínios exigem suas próprias Actions.
