---
id: enum-vehicle-event-type
title: Enum — VehicleEventType
description: Valores e invariantes propostos de VehicleEventType.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, veiculos
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Fueling` | `fueling` | Abastecimento; os dados de volume ficam na tabela específica de combustível |
| `Service` | `service` | Manutenção |
| `Repair` | `repair` | Reparo |
| `Inspection` | `inspection` | Inspeção |
| `Incident` | `incident` | Incidente |
| `Note` | `note` | Diário genérico; não implica despesa ou manutenção concluída |

## Regras e transições

Um evento pode ter vários itens realizados. `Fueling` tem dados próprios. `Note` registra qualquer acontecimento ou observação livre, sem efeito financeiro automático; despesas e pagamentos continuam no módulo financeiro. Task concluída não é automaticamente evento de serviço.
