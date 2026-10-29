---
id: migration-18-assets
title: Migration 18 — Bens e avaliações
description: Contrato lógico de tabelas, FKs, índices e conversão: bens e avaliações.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, patrimonio, componentes-tecnicos
source_refs: https://brasilapi.com.br/docs
updated: 2026-10-28
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 03, 06, 15. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

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

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
