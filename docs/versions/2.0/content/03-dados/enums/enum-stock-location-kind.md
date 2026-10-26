---
id: enum-stock-location-kind
title: Enum — StockLocationKind
description: Formato de local usado para navegação da árvore física.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, estoque
related: planejamento-2-0, enums, migrations, estoque-despensa, migration-22-stock
source_refs:
updated: 2026-10-26
---

## Contrato proposto

| Case | Valor persistido | Uso |
| --- | --- | --- |
| Property | property | Casa, escritório, garagem ou propriedade. |
| Room | room | Cômodo ou zona identificável. |
| Fixture | fixture | Armário, geladeira, prateleira, estante ou gaveta. |
| Container | container | Caixa, pote, mala ou compartimento. |
| Vehicle | vehicle | Porta-malas, porta-luvas ou compartimento móvel. |
| Other | other | Tipo ainda não coberto; nome continua personalizável. |

## Regras

Kind orienta ícone, filtro e sugestões de nome, sem exigir uma sequência fixa de níveis. A árvore é definida pelo parent_id. Um usuário pode criar locais com nomes livres; não criar enum por cômodo ou recipiente específico.

## Persistência

Aplicar em stock_locations.kind; manter os valores persistidos estáveis.
