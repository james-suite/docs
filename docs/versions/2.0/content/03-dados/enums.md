---
id: enums
title: Enums planejados
description: Catálogo de estados e conjuntos fechados propostos para o James 2.0.
type: development-hub
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, componentes-tecnicos
source_refs:
updated: 2026-10-28
---

## Critério

Usar enum para estados e famílias fechadas que mudam comportamento. Tags, categorias, nomes de combustível, tipos de relação com empresa, métodos específicos de pagamento e exercícios são catálogos, não enums gigantes. Arquivamento usa timestamp; atraso e quitação podem ser projeções, sem virar estado único que mistura várias dimensões.

Os cases são propostos; não são classes implementadas. Estados de funcionalidades futuras não habilitam recursos na interface. A implementação deve preservar os valores legados onde ainda tenham o mesmo significado.

## Catálogo

- [Enum — ClassificationMode](doc:enum-classification-mode) — `ai_user_settings`.

- [Enum — PartyKind](doc:enum-party-kind) — `parties`.
- [Enum — RecordKind](doc:enum-record-kind) — `records`.
- [Enum — TransactionKind](doc:enum-transaction-kind) — `financial_transactions`.
- [Enum — TransactionState](doc:enum-transaction-state) — `financial_transactions`.
- [Enum — TransactionPurpose](doc:enum-transaction-purpose) — `financial_transactions`.
- [Enum — ItemOwnership](doc:enum-item-ownership) — `financial_transaction_items`.
- [Enum — PaymentKind](doc:enum-payment-kind) — `financial_payments`.
- [Enum — PaymentStatus](doc:enum-payment-status) — `financial_payments`.
- [Enum — PaymentPayerType](doc:enum-payment-payer-type) — `financial_payments`.
- [Enum — LedgerAccountKind](doc:enum-ledger-account-kind) — `ledger_accounts`.
- [Enum — EntrySide](doc:enum-entry-side) — `ledger_postings`.
- [Enum — FinancialAccountType](doc:enum-financial-account-type) — `financial_accounts`.
- [Enum — InvoiceCycleState](doc:enum-invoice-cycle-state) — `credit_card_invoices`.
- [Enum — TagAssignmentOrigin](doc:enum-tag-assignment-origin) — `tag_assignments`.
- [Enum — SuggestionStatus](doc:enum-suggestion-status) — `classification_suggestions`.
- [Enum — RecurrenceFrequency](doc:enum-recurrence-frequency) — `financial_recurrences`.
- [Enum — SplitMode](doc:enum-split-mode) — `settlement_groups`.
- [Enum — ObligationDirection](doc:enum-obligation-direction) — `settlement_obligations`.
- [Enum — NotificationChannel](doc:enum-notification-channel) — `notification_deliveries`.
- [Enum — NotificationLevel](doc:enum-notification-level) — `notifications`.
- [Enum — DeliveryStatus](doc:enum-delivery-status) — `notification_delivery_attempts`.
- [Enum — AiRunStatus](doc:enum-ai-run-status) — `ai_runs`.
- [Enum — AiTask](doc:enum-ai-task) — `ai_runs`.
- [Enum — AiConnectionScope](doc:enum-ai-connection-scope) — `ai_connections`.
- [Enum — CaptureSource](doc:enum-capture-source) — `visits`.
- [Enum — JournalState](doc:enum-journal-state) — `journal_entries`.
- [Enum — VehicleEventType](doc:enum-vehicle-event-type) — `vehicle_events`.
- [Enum — VehicleFinancialLinkPurpose](doc:enum-vehicle-financial-link-purpose) — `vehicle_financial_links`.
- [Enum — FiscalCaptureMethod](doc:enum-fiscal-capture-method) — `fiscal_documents`.
- [Enum — FiscalExtractionStatus](doc:enum-fiscal-extraction-status) — `fiscal_documents, fiscal_document_extractions`.
- [Enum — TaskStatus](doc:enum-task-status) — `tasks`.
- [Enum — ChecklistItemStatus](doc:enum-checklist-item-status) — `checklist_items`.
- [Enum — ImportStatus](doc:enum-import-status) — `import_runs`.
- [Enum — ImportItemStatus](doc:enum-import-item-status) — `import_items`.

- [Enum — PaymentMethodFamily](doc:enum-payment-method-family) — `payment_methods`.
- [Enum — PriceObservationSource](doc:enum-price-observation-source) — `product_price_observations`.

- [Enum — TimePrecision](doc:enum-time-precision) — `financial_payments, journal_entries, vehicle_events`.

- [Enum — CalendarEventState](doc:enum-calendar-event-state) — `calendar_events`.
- [Enum — CalendarCommandAction](doc:enum-calendar-command-action) — `calendar_command_proposals`.

- [Enum — CardDebtDirection](doc:enum-card-debt-direction) — `card_charges`.
