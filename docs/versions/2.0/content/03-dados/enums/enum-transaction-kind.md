---
id: enum-transaction-kind
title: Enum — TransactionKind
description: Valores e invariantes propostos de TransactionKind.
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
| `Income` | `income` | Entrada econômica |
| `Expense` | `expense` | Saída econômica |

## Regras e transições

Direção econômica do documento; transferência e pagamento de fatura são tipos de operação financeira, não receita ou despesa nova.
