---
id: enum-transaction-purpose
title: Enum — TransactionPurpose
description: Valores e invariantes propostos de TransactionPurpose.
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
| `Standard` | `standard` | Operação usual |
| `Refund` | `refund` | Devolução vinculada |
| `Interest` | `interest` | Juros |
| `Fee` | `fee` | Tarifa |

## Regras e transições

O propósito descreve a natureza do documento. Liquidação de adiantamento entre pessoas/empresas pertence a `settlement_obligations` e `settlement_payments`, não a um propósito `Reimbursement` nem a uma tag. `Refund` é devolução comercial ligada à compra original; não usar para pagamento de acerto.
