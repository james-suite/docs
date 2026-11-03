---
id: enum-obligation-direction
title: Enum — ObligationDirection
description: Valores e invariantes propostos de ObligationDirection.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, acertos
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Receivable` | `receivable` | Tenho a receber |
| `Payable` | `payable` | Tenho a pagar |

## Regras e transições

Liquidações ficam em tabela própria. Não guardar direção e sinal negativo simultaneamente; valor positivo e direção explícita.
