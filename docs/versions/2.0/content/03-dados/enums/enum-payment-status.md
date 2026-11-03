---
id: enum-payment-status
title: Enum — PaymentStatus
description: Valores e invariantes propostos de PaymentStatus.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Draft` | `draft` | Rascunho |
| `Scheduled` | `scheduled` | Programado |
| `Posted` | `posted` | Efetivado |
| `Reversed` | `reversed` | Integralmente estornado |
| `Cancelled` | `cancelled` | Cancelado antes da efetivação |

## Regras e transições

draft → scheduled/posted/cancelled; scheduled → posted/cancelled; posted → reversed por operação inversa. Estorno parcial mantém posted e total estornado derivado. Lançamentos originais nunca desaparecem.
