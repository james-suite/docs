---
id: enum-item-ownership
title: Enum — ItemOwnership
description: Valores e invariantes propostos de ItemOwnership.
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
| `Personal` | `personal` | Consumo próprio |
| `ThirdParty` | `third_party` | Valor de terceiro |

## Regras e transições

A divisão permite separar despesa pessoal e adiantamento. Itens de terceiro devem explicar sua relação com cotas/obrigações; não inferir por nome de tag.
