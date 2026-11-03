---
id: enum-ai-run-status
title: Enum — AiRunStatus
description: Valores e invariantes propostos de AiRunStatus.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, ia-e-memoria
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Queued` | `queued` | Reservada e na fila |
| `Running` | `running` | Em execução |
| `Succeeded` | `succeeded` | Resultado validado |
| `Failed` | `failed` | Falhou |
| `Cancelled` | `cancelled` | Cancelada |

## Regras e transições

queued → running/cancelled; running → succeeded/failed/cancelled. Timeout ambíguo exige reconciliação de uso; cancelamento não garante ausência de cobrança no provedor.
