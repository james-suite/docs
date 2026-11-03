---
id: enum-checklist-item-status
title: Enum — ChecklistItemStatus
description: Valores e invariantes propostos de ChecklistItemStatus.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, veiculos
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Pending` | `pending` | Pendente |
| `Done` | `done` | Conferido |
| `NotApplicable` | `not_applicable` | Não aplicável |

## Regras e transições

Não aplicável exige justificativa quando o modelo requer. Reabrir preserva eventos anteriores; alterar template não altera instância histórica.
