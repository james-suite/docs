---
id: enum-ledger-account-kind
title: Enum — LedgerAccountKind
description: Valores e invariantes propostos de LedgerAccountKind.
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
| `Asset` | `asset` | Ativo |
| `Liability` | `liability` | Passivo |
| `Expense` | `expense` | Despesa |
| `Income` | `income` | Receita |
| `Equity` | `equity` | Contrapartida patrimonial |

## Regras e transições

Proposta do livro interno condicionada ao protótipo. Define saldo natural e contrapartidas; não corresponde às categorias livres do usuário.
