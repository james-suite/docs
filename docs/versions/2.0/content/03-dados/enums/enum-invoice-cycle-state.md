---
id: enum-invoice-cycle-state
title: Enum — InvoiceCycleState
description: Valores e invariantes propostos de InvoiceCycleState.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, cartoes
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Open` | `open` | Ciclo aberto |
| `Closed` | `closed` | Ciclo fechado |
| `Cancelled` | `cancelled` | Ciclo cancelado |

## Regras e transições

open → closed; reabertura controlada gera auditoria. Cancelamento exige ausência de efeitos financeiros ou reversão compatível. Quitação e atraso são calculados em dimensões separadas.
