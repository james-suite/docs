---
id: enum-card-debt-direction
title: Enum — CardDebtDirection
description: Valores e invariantes propostos de CardDebtDirection.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, cartoes
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Increase` | `increase` | Aumenta dívida do cartão |
| `Decrease` | `decrease` | Crédito/devolução que reduz dívida |

## Regras

amount_minor é positivo; direção define efeito no passivo e nas parcelas. Increase/Decrease não são os lados Debit/Credit do livro interno. Devolução aponta à compra original, respeita valor remanescente e pode afetar outro ciclo.
