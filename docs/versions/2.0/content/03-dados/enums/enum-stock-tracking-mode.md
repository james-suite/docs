---
id: enum-stock-tracking-mode
title: Enum — StockTrackingMode
description: Modalidade de saldo usada por um item do estoque.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, estoque
related: planejamento-2-0, enums, migrations, estoque-despensa, migration-22-stock, migration-23-stock-operations
source_refs:
updated: 2026-10-26
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| Aggregate | aggregate | Quantidade consolidada por item/local, sem lote individual. |
| Lot | lot | Saldo segregado por recebimento/lote e datas opcionais. |
| Individual | individual | Cada unidade tem identidade e histórico físico próprios. |

## Regras

Aggregate e Lot entram na base inicial da Migration 22. Individual depende de stock_units e da Migration 23. Um modo futuro não habilita UI ou gravação antes da tabela e das invariantes correspondentes.

Troca de modalidade exige uma operação revisável: criação de lotes para o saldo existente ou cadastro de unidades individuais/contagem. Nunca alterar a interpretação histórica de movimentos antigos.

## Persistência

Cast string em stock_items.tracking_mode. Campos e restrições detalhados em [Migration 22](doc:migration-22-stock) e [Migration 23](doc:migration-23-stock-operations).
