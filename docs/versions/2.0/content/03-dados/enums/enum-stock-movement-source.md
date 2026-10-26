---
id: enum-stock-movement-source
title: Enum — StockMovementSource
description: Procedência de um evento que altera a quantidade física.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, estoque
related: planejamento-2-0, enums, migrations, estoque-despensa, migration-22-stock
source_refs:
updated: 2026-10-26
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| Manual | manual | A pessoa registrou a entrada, saída ou transferência no site. |
| FinancialItem | financial_item | Entrada confirmada a partir de linha financeira vinculada. |
| InitialCount | initial_count | Saldo inicial informado ao começar a usar o módulo. |
| Gift | gift | Entrada recebida como presente/amostra. |
| Import | import | Entrada incluída após revisão de arquivo suportado. |
| Stocktake | stocktake | Ajuste aprovado ao concluir contagem física. |
| Correction | correction | Movimento compensatório ligado ao evento corrigido. |

## Regras

Origem informa procedência e não substitui StockMovementType. FinancialItem exige referência financeira válida e quantidade coerente; Stocktake exige sessão concluída; Correction aponta ao movimento original. Dados importados não ganham lote/local inventado.

## Persistência

Aplicar em stock_movements.source. A fonte não implica automaticamente despesa, ajuste contábil ou confiança no produto.
