---
id: migration-18-assets
title: Migration 18 — Bens e avaliações
description: Contrato lógico de tabelas, FKs, índices e conversão: bens e avaliações.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, patrimonio, documentos-arquivos, migration-22-documents, componentes-tecnicos
source_refs: https://brasilapi.com.br/docs
updated: 2026-11-03
---

## `assets`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, name, category_code, serial_number` | Bem e classificação de catálogo pessoal. |
| `acquired_on, disposed_on, acquisition_minor, currency` | Posse e custo conhecidos. |
| `party_id, purchase_transaction_id, vehicle_id` | Origem/compra e vínculo opcional único a `vehicles`; veículo sem linha patrimonial continua válido no módulo de veículos. |
| `archived_at, notes` | Histórico preservado. |

## `asset_valuations`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, asset_id, valued_on, amount_minor, currency, source` | Avaliação datada com fonte e valor não negativo. Uma cotação FIPE confirmada é uma fonte possível. |
| `source_reference, notes` | Referência externa como código/mês da tabela FIPE e critério/observação; não gerar entrada de caixa. |

## Integridade, índices e comportamento

- UNIQUE(space_id, vehicle_id) quando não nulo. Um veículo pode não ser um bem cadastrado; quando é, não pode ser contado duas vezes no patrimônio.
- Soma patrimonial informa data/base de avaliação. Investimento financeiro já contado não vira outro bem equivalente.
- Alterar preço estimado não cria receita/despesa ou lançamento bancário.

## Conversão do James 1.x

Módulo novo. Transações antigas podem ser associadas a aquisição sem novo movimento de caixa.

## Componentes previstos

RegisterAsset; RecordAssetValuation; DisposeAsset; NetWorthQuery com regra de não duplicação.
