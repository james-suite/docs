---
id: enum-financial-account-type
title: Enum — FinancialAccountType
description: Valores e invariantes propostos de FinancialAccountType.
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
| `Checking` | `checking` | Conta corrente |
| `Investment` | `investment` | Investimento |
| `Wallet` | `wallet` | Carteira |
| `Savings` | `savings` | Poupança |

## Regras e transições

Preservar os três valores existentes no 1.x. Tipo não controla arquivamento; archived_at é independente.
