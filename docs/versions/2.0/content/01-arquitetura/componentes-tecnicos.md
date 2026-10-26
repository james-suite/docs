---
id: componentes-tecnicos
title: Componentes técnicos planejados
description: Models, Actions, Policies, casts, Jobs e componentes de interface necessários por domínio.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: modelo-de-dados, migrations, enums, ui-ux-2-0, fases-2-0
source_refs:
updated: 2026-10-26
---

## Organização da implementação

CRUD simples segue o padrão vigente do projeto. Operações com várias entidades, concorrência, importação ou transporte externo justificam Actions/Services próprias. Não criar Repository, interface ou Service genérico para cada tabela. Interfaces são úteis nas fronteiras de provedor de IA, importador e transporte.

## Models e operações

| Área | Models centrais | Operações e consultas necessárias |
| --- | --- | --- |
| Identidade | User, Space | PersonalSpaceContext, provisionar espaço pessoal, consultas por proprietário. |
| Pessoas | Party, PartyIdentifier, PartyContactMethod, PartyGroup, PartyRelationship | Salvar, buscar alias/identificador, fusão revisada e arquivamento. |
| Classificação | Tag, TagName, TagAssignment, FinancialCategory, ClassificationRule | SaveTagsForRecord, MergeTags, ApplyClassificationRules, classificação por item. |
| Financeiro | FinancialAccount, FinancialTransaction, FinancialTransactionItem, FiscalDocument, FiscalDocumentExtraction, FinancialPayment, PaymentAllocation | Captura/revisão fiscal, SaveTransaction, PostPayment por pagador, ReversePayment, TransferFunds, consultas de caixa e consumo. |
| Livro interno proposto | LedgerAccount, LedgerJournal, LedgerPosting | JournalBuilder e BalanceQuery; gate de protótipo antes da aprovação. |
| Cartões | CreditCard, CardRuleVersion, CreditCardInvoice, CardCharge, CardInstallment, InvoicePaymentAllocation | Resolver/fechar ciclo, gerar parcelas, pagar/estornar fatura e calcular limite. |
| Recorrências | FinancialRecurrence, RecurrenceTemplate, RecurrenceOccurrence | Prévia de datas, materialização idempotente, pausa e exceções. |
| Acertos | SettlementGroup, SplitParticipant, SplitShare, SplitContribution, SettlementObligation, SettlementPayment | Rateio sem parcela própria, contribuições, liquidação parcial e projeção de saldos. |
| Notificações | Notification, NotificationPreference, NotificationEndpoint, NotificationDelivery, NotificationDeliveryAttempt | Outbox, digest, reenvio, resolução/adiamento e deduplicação durável. |
| IA | AiConnection, AiRun, ClassificationSuggestion, AiUsageEntry, AiBudgetPeriod | Seleção de conexão autorizada, reserva de orçamento, sugestão/revisão e falhas. |
| Captura/diário | Place, Visit, CaptureDevice, JournalEntry, JournalEntryRevision | Check-in, captura por token restrito, diário versionado e vínculos. |
| Veículos | Vehicle, VehicleEvent, VehicleEventItem, FuelEvent, VehicleFinancialLink, OdometerReading, MaintenancePlan | Leitura de nota, vínculo sem duplicar custos, diário genérico, serviços reais com vários itens, consumo, correção de odômetro e vencimento por km/data. |
| Organização | Task, Checklist, ChecklistItem, CalendarEvent, CalendarSeries | Tarefas, checklist histórico, exceções de série e lembretes. |
| Patrimônio/saúde | Asset, AssetValuation, Exercise, WorkoutRoutine, WorkoutSession, WorkoutSet, HealthObservation | Evolução patrimonial sem duplicação; treino real e unidades de medição. |
| Portabilidade | DataExport no 1.x; ImportRun, ImportItem, LegacyIdMap no 2.0 | Gerar arquivo, analisar upload, prévia, confirmar na própria conta, reconciliar e limpar staging. |

## Casts, valores e helpers

Money e Quantity precisam de operações exatas e arredondamento explícito; usar biblioteca compatível ou objetos pequenos em vez de espalhar casts para float. Campos de enum recebem cast próprio. Datas locais, instantes UTC e fuso IANA devem ser distinguidos no domínio.

Aproveitar do SGE helpers de documentos brasileiros, telefone, moeda/data e normalização quando o contrato for realmente igual. Validar a API dos pacotes em 2027; copiar nomes de métodos antigos de skill/documentação não garante compatibilidade.

## Policies e validação

Cada recurso tem Policy por proprietário, além das verificações de estado. Seletores, relacionamentos, anexos, revisões, busca, relatórios, auditoria e downloads recebem o mesmo escopo. Administrador da instalação administra contas e operação; isso não concede leitura de diário ou finanças dos usuários na interface.

Form Requests/validação Livewire aceitam somente dados do formulário; space_id, autoria e idempotência são resolvidos no servidor. Revalidar dentro da transação os registros que podem ter sido arquivados ou alterados depois de o formulário abrir.

## Jobs e processos agendados

Gerar exportação, validar/importar pacote, enviar notificações, materializar recorrências e chamar IA em filas. Job carrega IDs de domínio e de conexão, nunca uma API key serializada. Definir timeout, backoff, failed() sanitizado, limites externos e recuperação do estado após interrupção. UNIQUE no banco continua sendo a garantia final de idempotência.

## Componentes de UI

TagPicker com criação inline a partir de `select-create` do SGE; FiscalDocumentCapture com revisão campo a campo; PaymentEditor para vários meios/datas e pagadores; SplitEditor com lista explícita de consumidores/pagadores; InvoicePaymentsHistory; NotificationInbox; ImportWizard; VehicleServiceEditor com vários itens e data real; VehicleDiaryEditor; PrivateFileUpload; AuditHistory.

Os nomes são propostas de responsabilidades, não classes já criadas. Cada componente deve ter loading, validação, teclado e retorno ao contexto anterior. Flux 2 gratuito fornece a base; componentes próprios cobrem o domínio sem depender de recursos Pro.

## Qualidade

Factories por estado de domínio e seeds com códigos estáveis, sem IDs mágicos de tags. Testes relevantes cobrem autorização cruzada, somas, concorrência, estornos, importação e falhas externas. Não criar testes que só repitam textos de documentação ou classes CSS.
