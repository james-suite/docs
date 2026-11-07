---
id: enum-record-kind
title: Enum — RecordKind
description: Valores e invariantes propostos de RecordKind.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, modelo-de-dados
source_refs:
updated: 2026-11-07
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Party` | `party` | Pessoa ou empresa |
| `Transaction` | `transaction` | Transação |
| `TransactionItem` | `transaction_item` | Item de transação |
| `FinancialAdjustment` | `financial_adjustment` | Ajuste da transação com origem e classificação próprias |
| `Payment` | `payment` | Pagamento |
| `Card` | `card` | Cartão |
| `Invoice` | `invoice` | Fatura |
| `FinancialRecurrence` | `financial_recurrence` | Regra de recorrência financeira |
| `SettlementGroup` | `settlement_group` | Rateio |
| `SettlementObligation` | `settlement_obligation` | Obrigação de acerto |
| `SettlementPayment` | `settlement_payment` | Liquidação de acerto |
| `Vehicle` | `vehicle` | Veículo |
| `VehicleEvent` | `vehicle_event` | Evento do veículo |
| `Place` | `place` | Lugar ou estabelecimento |
| `Visit` | `visit` | Visita |
| `JournalEntry` | `journal_entry` | Entrada de diário |
| `Task` | `task` | Tarefa |
| `CalendarEvent` | `calendar_event` | Compromisso |
| `Asset` | `asset` | Bem |
| `AssetValuation` | `asset_valuation` | Avaliação patrimonial |
| `WorkoutSession` | `workout_session` | Sessão de treino |
| `HealthObservation` | `health_observation` | Medição de saúde |
| `Product` | `product` | Produto do catálogo pessoal |
| `ProductListing` | `product_listing` | Cadastro comercial pessoal de produto |
| `Document` | `document` | Documento pessoal e ponto de entrada para suas versões e vínculos |

## Regras e transições

Registro de tipos mantido em código. Resolver somente Models permitidos, nunca nome de classe recebido do cliente. Um tipo novo exige integração com busca, Policy, exportação e exclusão.
