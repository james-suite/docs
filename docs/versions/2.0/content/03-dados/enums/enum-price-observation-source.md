---
id: enum-price-observation-source
title: Enum — PriceObservationSource
description: Origem de uma observação de preço individual do catálogo pessoal.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, produtos-precos, migration-21-products
source_refs:
updated: 2026-10-26
---

## Contrato proposto

Enum string proposta; ainda não existe classe PHP no 2.0.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| FiscalExtraction | fiscal_extraction | Linha criada a partir de documento fiscal revisado. |
| ManualEntry | manual_entry | Linha financeira preenchida manualmente e confirmada. |
| LegacyImport | legacy_import | Observação criada após confirmação explícita na importação. |
| Correction | correction | Nova observação que corrige uma anterior sem apagar sua proveniência. |

## Regras

A origem informa como a observação foi produzida, não sua qualidade. Toda observação aponta para uma linha financeira e a captura fiscal mantém também seu próprio método e processador.

## Persistência

Aplicar em product_price_observations conforme Migration 21.
