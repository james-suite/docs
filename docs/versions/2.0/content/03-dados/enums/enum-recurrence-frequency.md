---
id: enum-recurrence-frequency
title: Enum — RecurrenceFrequency
description: Valores e invariantes propostos de RecurrenceFrequency.
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
| `Daily` | `daily` | Diária |
| `Weekly` | `weekly` | Semanal |
| `Monthly` | `monthly` | Mensal |
| `Yearly` | `yearly` | Anual |

## Regras e transições

Intervalo inteiro positivo complementa a frequência. Preservar âncora original em dias inexistentes no mês; não acumular deriva de data.
