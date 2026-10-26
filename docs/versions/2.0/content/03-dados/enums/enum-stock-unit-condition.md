---
id: enum-stock-unit-condition
title: Enum — StockUnitCondition
description: Condição registrada de uma unidade física individual.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, estoque
related: planejamento-2-0, enums, migrations, estoque-despensa, migration-23-stock-operations
source_refs:
updated: 2026-10-26
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| New | new | Sem uso ou em estado novo conforme informado. |
| Good | good | Em condição de uso. |
| Worn | worn | Desgaste visível que não impede o uso. |
| NeedsRepair | needs_repair | Exige reparo antes de novo uso. |
| Damaged | damaged | Danificado; manter visível para decisão. |
| Disposed | disposed | Destinação registrada, sem saldo físico disponível. |

## Regras

Condição descreve a unidade; não substitui movimento de perda, descarte, checkout ou retorno. Marcar Disposed requer evento de saída compatível e nunca apaga o histórico.

## Persistência

Aplicar em stock_units.condition conforme Migration 23.
