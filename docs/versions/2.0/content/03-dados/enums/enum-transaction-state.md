---
id: enum-transaction-state
title: Enum — TransactionState
description: Valores e invariantes propostos de TransactionState.
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
| `Confirmed` | `confirmed` | Confirmado |
| `Cancelled` | `cancelled` | Cancelado |

## Regras e transições

draft → confirmed ou cancelled. confirmed → cancelled exige ausência de pagamentos efetivos e de vínculos irreversíveis; se necessário, reverter tecnicamente os pagamentos antes. Compra paga que recebeu devolução comercial permanece confirmed com documento de devolução relacionado. Não esconder documento pago do relatório apenas alterando o estado. Quitação é calculada separadamente.
