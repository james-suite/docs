---
id: enum-vehicle-financial-link-purpose
title: Enum — VehicleFinancialLinkPurpose
description: Finalidade do vínculo entre uma linha financeira e um veículo.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, veiculos, financas
source_refs:
updated: 2026-10-26
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Fuel` | `fuel` | Linha de combustível ligada ao evento de abastecimento. |
| `PurchaseForVehicle` | `purchase_for_vehicle` | Item destinado ao veículo, sem afirmar que foi instalado ou usado. |
| `ServiceCost` | `service_cost` | Custo ligado a um evento de serviço realmente registrado. |
| `Other` | `other` | Relação financeira contextual sem semântica de manutenção específica. |

## Regras

Finalidade não muda valor, pagamento ou saldo. `PurchaseForVehicle` não exige `vehicle_event_id`; preenchê-lo depois registra em qual evento o item foi realmente usado. `Fuel` exige evento de abastecimento e item financeiro se preço for conhecido; `ServiceCost` exige evento real. A relação pode alocar só parte da quantidade/valor da linha, sem somar mais que o original.

## Persistência

Aplicar em `vehicle_financial_links` conforme [Migration 15](doc:migration-15-vehicles). Persistir string com cast Eloquent e restrição de valores quando apropriado.

## Checklist

- [ ] Revisar finalidade e transições no fluxo de [veículos](doc:veiculos).
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Validar espaço, linha financeira, evento e quantidade associada.
