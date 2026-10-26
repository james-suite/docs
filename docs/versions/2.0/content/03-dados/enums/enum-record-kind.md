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
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Party` | `party` | Pessoa ou empresa |
| `Transaction` | `transaction` | Transação |
| `TransactionItem` | `transaction_item` | Item de transação |
| `Payment` | `payment` | Pagamento |
| `Card` | `card` | Cartão |
| `Invoice` | `invoice` | Fatura |
| `FinancialRecurrence` | `financial_recurrence` | Regra de recorrência financeira |
| `SettlementGroup` | `settlement_group` | Rateio |
| `SettlementObligation` | `settlement_obligation` | Obrigação de acerto |
| `SettlementPayment` | `settlement_payment` | Liquidação de acerto |
| `Vehicle` | `vehicle` | Veículo |
| `VehicleEvent` | `vehicle_event` | Evento do veículo |
| `FuelEvent` | `fuel_event` | Abastecimento |
| `Visit` | `visit` | Visita |
| `JournalEntry` | `journal_entry` | Entrada de diário |
| `Task` | `task` | Tarefa |
| `CalendarEvent` | `calendar_event` | Compromisso |
| `Asset` | `asset` | Bem |
| `AssetValuation` | `asset_valuation` | Avaliação patrimonial |
| `WorkoutSession` | `workout_session` | Sessão de treino |
| `MetricObservation` | `metric_observation` | Medição |

## Regras e transições

Registro de tipos mantido em código. Resolver somente Models permitidos, nunca nome de classe recebido do cliente. Um tipo novo exige integração com busca, Policy, exportação e exclusão.

## Persistência

Aplicar em `records` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:modelo-de-dados) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
