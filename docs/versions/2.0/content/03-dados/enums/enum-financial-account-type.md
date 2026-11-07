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
updated: 2026-11-07
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Checking` | `checking` | Conta corrente |
| `Investment` | `investment` | Investimento |
| `Wallet` | `wallet` | Dinheiro físico |
| `Savings` | `savings` | Poupança |

## Regras e transições

O rótulo de Wallet passa a Dinheiro físico. Preservar o valor persistido `wallet`, os saldos e a correspondência de IDs de origem no mapa de importação. O destino usa seus próprios IDs locais. Tipo não controla arquivamento; archived_at é independente.
