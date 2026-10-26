---
id: enum-stock-movement-type
title: Enum — StockMovementType
description: Efeito físico produzido por um movimento de estoque.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, estoque
related: planejamento-2-0, enums, migrations, estoque-despensa, migration-22-stock
source_refs:
updated: 2026-10-26
---

## Contrato proposto

| Case | Valor persistido | Efeito |
| --- | --- | --- |
| Receipt | receipt | Entrada positiva por compra, presente, estoque inicial ou entrada manual. |
| Consumption | consumption | Saída positiva de quantidade consumida/usada. |
| Transfer | transfer | Linhas de saída e entrada iguais em locais diferentes; saldo total não muda. |
| Adjustment | adjustment | Correção positiva ou negativa, sempre com motivo e revisão. |
| Loss | loss | Saída por perda, dano, expiração ou descarte, com motivo. |
| Checkout | checkout | Saída temporária para custódia/ empréstimo. |
| Return | return | Entrada de unidade/quantidade devolvida ao estoque. |

## Regras

Delta de movimento é assinado: entrada positiva e saída negativa. Receipt/Return só aceitam entrada; Consumption/Loss/Checkout só aceitam saída; Transfer exige soma zero; Adjustment aceita qualquer sinal. Convenção e validação ficam na Action, além de constraints por linha e testes transacionais.

Devolução para fornecedor é uma saída Loss ligada a um motivo comercial e eventual operação financeira própria. Return é reservado ao retorno físico de empréstimo/custódia.

## Persistência

Aplicar em stock_movements.movement_type; valores persistidos são estáveis e não traduzidos. Consulte [Migration 22](doc:migration-22-stock).
