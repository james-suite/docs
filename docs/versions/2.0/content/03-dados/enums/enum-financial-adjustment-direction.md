---
id: enum-financial-adjustment-direction
title: Enum — FinancialAdjustmentDirection
description: Direção de um desconto ou acréscimo global conciliado na transação.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: enums, financas, migration-06-transactions
source_refs:
updated: 2026-11-07
---

## Contrato proposto

| Case | Valor persistido | Efeito |
| --- | --- | --- |
| `Discount` | `discount` | Reduz o total pelos centavos informados |
| `Surcharge` | `surcharge` | Aumenta o total pelos centavos informados |

Persistir amount_minor positivo. Alocações transferem a atribuição às linhas e conservam o total financeiro do ajuste. Validar a soma sob transação e bloqueio.
