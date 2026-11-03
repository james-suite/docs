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
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Fuel` | `fuel` | Linha de combustível ligada ao evento de abastecimento. |
| `PurchaseForVehicle` | `purchase_for_vehicle` | Item destinado ao veículo, sem afirmar que foi instalado ou usado. |
| `ServiceCost` | `service_cost` | Custo ligado a um evento de serviço realmente registrado. |
| `Other` | `other` | Relação financeira contextual sem semântica de manutenção específica. |

## Regras

Finalidade não muda valor, pagamento ou saldo. `PurchaseForVehicle` não exige `vehicle_event_id`; preenchê-lo depois registra em qual evento o item foi realmente usado. `Fuel` exige evento de abastecimento e item financeiro se preço for conhecido; `ServiceCost` exige evento real. A relação pode alocar só parte da quantidade/valor da linha, sem somar mais que o original.
