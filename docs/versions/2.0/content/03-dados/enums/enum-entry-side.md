---
id: enum-entry-side
title: Enum — EntrySide
description: Valores e invariantes propostos de EntrySide.
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
| `Debit` | `debit` | Débito |
| `Credit` | `credit` | Crédito |

## Regras e transições

Valor de posting sempre positivo. O lado determina o sinal. Em cada lote e moeda, soma de débitos igual à de créditos.
