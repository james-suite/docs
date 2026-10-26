---
id: enum-storage-environment
title: Enum — StorageEnvironment
description: Ambiente descritivo associado a um local de armazenamento.
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
| Ambient | ambient | Armazenamento em ambiente comum. |
| Refrigerated | refrigerated | Refrigeração. |
| Frozen | frozen | Congelamento. |
| TemperatureControlled | temperature_controlled | Ambiente com controle específico definido pela pessoa. |
| Other | other | Informação ainda não modelada. |

## Regras

É metadado informativo do local. Não afirma que um sensor mediu a temperatura, não valida automaticamente uma faixa segura e não substitui orientações do fabricante.

## Persistência

Aplicar em stock_locations.environment; pode ser nulo se desconhecido ou irrelevante.
