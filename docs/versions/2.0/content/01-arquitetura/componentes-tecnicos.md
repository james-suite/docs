---
id: componentes-tecnicos
title: Componentes técnicos planejados
description: Models, Actions, Policies, casts, Jobs e componentes de interface necessários por domínio.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: modelo-de-dados, migrations, enums, ui-ux-2-0, fases-2-0
source_refs: https://brasilapi.com.br/docs
updated: 2026-10-29
---

## Organização da implementação

CRUD simples segue o padrão vigente do projeto. Operações com várias entidades, concorrência, importação ou transporte externo justificam Actions/Services próprias. Não criar Repository, interface ou Service genérico para cada tabela. Interfaces são úteis nas fronteiras de provedor de IA, importador e transporte.

## Models e operações

| Área | Models centrais | Operações e consultas necessárias |
| --- | --- | --- |
| Identidade | User, Space, UserPreference | PersonalSpaceContext, provisionar espaço pessoal, preferência de captura de localização no salvamento e consultas por proprietário. |
| Pessoas | Party, PartyIdentifier, PartyContactMethod, PartyGroup, PartyRelationship | Salvar, buscar alias/identificador, fusão revisada e arquivamento. |
| Classificação | Tag, TagName, TagAssignment, FinancialCategory, ClassificationRule | SaveTagsForRecord, MergeTags, ApplyClassificationRules, classificação por item. |
| Financeiro | FinancialAccount, FinancialTransaction, FinancialTransactionItem, FiscalDocument, FiscalDocumentExtraction, FinancialPayment, PaymentAllocation | Captura manual/por texto e adaptador fiscal por UF com fallback revisável; DraftTransactionFromText, ReviewAiTransactionDraft, vínculo de emitente/filial, PostPayment por pagador, ReversePayment, TransferFunds, consultas de caixa e consumo. |
| Produtos e preços | Product, ProductIdentifier, ProductListing, ProductListingIdentifier, ProductAlias, ProductCodeProfile, ProductCodeDecoding, MeasurementUnit, ProductPriceObservation, PriceBasket, PriceBasketItem | Separar produto comparável do catálogo local da loja; resolver códigos/descrições por filial, empresa e vigência; decodificar etiqueta variável por perfil determinístico validado; registrar observação de linha confirmada; normalizar unidade; consultar histórico e cesta pessoal com cobertura. |
| Livro interno proposto | LedgerAccount, LedgerJournal, LedgerPosting | JournalBuilder e BalanceQuery; gate de protótipo antes da aprovação. |
| Cartões | CreditCard, CardRuleVersion, CreditCardInvoice, CardCharge, CardInstallment, InvoicePaymentAllocation | Resolver/fechar ciclo, gerar parcelas, pagar/estornar fatura e calcular limite. |
| Recorrências | FinancialRecurrence, RecurrenceTemplate, RecurrenceOccurrence | Prévia de datas, materialização idempotente, pausa e exceções. |
| Acertos | SettlementGroup, SplitParticipant, SplitShare, SplitContribution, SettlementObligation, SettlementPayment | Rateio sem parcela própria, contribuições, liquidação parcial e projeção de saldos. |
| Notificações | Notification, NotificationPreference, NotificationEndpoint, NotificationDelivery, NotificationDeliveryAttempt | Outbox, digest, reenvio, resolução/adiamento e deduplicação durável. |
| IA | AiConnection, AiRun, ClassificationSuggestion, CalendarCommandProposal, AiUsageEntry, AiBudgetPeriod | Seleção de conexão autorizada, reserva de orçamento, sugestão/revisão de tags, comandos de agenda e captura tipada de transação por texto, se aprovados. |
| Captura/diário | Place, PlaceIdentifier, Visit, JournalEntry, JournalEntryRevision, ActivityLocationContext | Filial/endereço, identificadores empresariais, check-in pelo site, contexto geográfico por ação auditada, diário versionado e vínculos. |
| Veículos | Vehicle, VehicleEvent, VehicleEventItem, FuelEvent, VehicleFinancialLink, OdometerReading, MaintenancePlan | Lançamento independente de odômetro, leitura de nota, resolução do posto/filial, vínculo sem duplicar custos, diário genérico, serviços reais com vários itens, consumo, correção de odômetro e vencimento por km/data. |
| Organização | Task, Checklist, ChecklistItem, CalendarEvent, CalendarSeries, CalendarFeedToken, CalendarConnection, CalendarEventExternalLink | Tarefas, checklist histórico, exceções de série, lembretes, feed privado `.ics` e conexão bidirecional opcional posterior via CalDAV/Google. |
| Patrimônio/saúde | Asset, AssetValuation, Exercise, WorkoutRoutine, WorkoutSession, WorkoutSet, HealthObservation | Evolução patrimonial sem duplicação; treino real e unidades de medição. |
| Dados públicos do Brasil | BrasilApiReferenceLookup | Consultas sob demanda de CNPJ, CEP, bancos, NCM, FIPE e feriados; registrar origem e data de consulta quando a resposta alimentar um cadastro. |
| Portabilidade | DataExport no 1.x; ImportRun, ImportItem, LegacyIdMap no 2.0 | Gerar arquivo, analisar upload, prévia, confirmar na própria conta, reconciliar e limpar staging. |

## Casts, valores e helpers

Money e Quantity precisam de operações exatas e arredondamento explícito; usar biblioteca compatível ou objetos pequenos em vez de espalhar casts para float. Campos de enum recebem cast próprio. Datas locais, instantes UTC e fuso IANA devem ser distinguidos no domínio.

Aproveitar do SGE helpers de documentos brasileiros, telefone, moeda/data e normalização quando o contrato for realmente igual. Validar a API dos pacotes em 2027; copiar nomes de métodos antigos de skill/documentação não garante compatibilidade.

## Policies e validação

Cada recurso tem Policy por proprietário, além das verificações de estado. Seletores, relacionamentos, anexos, revisões, busca, relatórios, auditoria e downloads recebem o mesmo escopo. Administrador da instalação administra contas e operação; isso não concede leitura de diário ou finanças dos usuários na interface.

Form Requests/validação Livewire aceitam somente dados do formulário; space_id, autoria e idempotência são resolvidos no servidor. Revalidar dentro da transação os registros que podem ter sido arquivados ou alterados depois de o formulário abrir.

## Jobs e processos agendados

Gerar exportação, validar/importar pacote, enviar notificações, materializar recorrências, sincronizar calendários externos e chamar IA em filas. Job carrega IDs de domínio e de conexão, nunca uma API key serializada. Definir timeout, backoff, failed() sanitizado, limites externos e recuperação do estado após interrupção. UNIQUE no banco continua sendo a garantia final de idempotência.

## Componentes de UI

TagPicker com criação inline a partir de `select-create` do SGE; FiscalDocumentCapture com revisão campo a campo; PaymentEditor para vários meios/datas e pagadores; SplitEditor com lista explícita de consumidores/pagadores; InvoicePaymentsHistory; NotificationInbox; ImportWizard; VehicleServiceEditor com vários itens e data real; VehicleDiaryEditor; PrivateFileUpload; AuditHistory.

Incluir também FiscalIssuerPlaceMatcher, AiTransactionComposer, ProductMatcher com identificação por empresa/estabelecimento e criação revisável, ProductListingReview, VariableMeasureCodeReview com peso/preço/total e origem visíveis, PriceHistory, PersonalPriceBasket, BrowserLocationCapture no salvamento, ActivityLocationMap, OdometerQuickEntry independente do abastecimento e CalendarCommandComposer com revisão de alvo/alterações. A captura geográfica deve associar-se por correlação ao evento de auditoria; o mapa usa tiles de provedor configurável e respeita retenção da localização. O decoder de etiqueta deve ser determinístico; só perfis testados com etiquetas reais podem preencher candidatos. Os nomes são propostas de responsabilidades, não classes já criadas. Cada componente deve ter loading, validação, teclado e retorno ao contexto anterior. Flux 2 gratuito fornece a base; componentes próprios cobrem o domínio sem depender de recursos Pro.

## Consultas de referência pela BrasilAPI

Usos propostos, sempre ligados a uma ação ou formulário do usuário:

| Domínio | Consulta | Rota (sufixo da documentação) | Uso no James |
| --- | --- | --- | --- |
| Pessoas e empresas | CNPJ | `/cnpj/v1/{cnpj}` | Sugerir razão social, nome de uso e dados do estabelecimento para preencher Party/Place. |
| Endereços | CEP | `/cep/v1/{cep}`, opcionalmente `/cep/v2/{cep}` | Preencher endereço de lugar; coordenadas da versão 2 são apenas uma sugestão aproximada associada ao CEP. |
| Finanças | Bancos | `/banks/v1`, `/banks/v1/{code}` | Pesquisar instituição e código ao cadastrar conta financeira; não conecta à conta nem consulta saldo. |
| Notas e produtos | NCM | `/ncm/v1/{code}` ou `/ncm/v1?search=...` | Conferir código, descrição e vigência do NCM quando ele já vier na linha fiscal. Não identifica sozinho o produto comercial nem cria tags. |
| Veículos e patrimônio | FIPE | `/fipe/detalhes/v1/...`, `/fipe/preco/v1/{fipeCode}` | Ajudar a selecionar marca/modelo/ano e consultar valor por código e mês de referência. |
| Agenda | Feriados | `/feriados/v1/{ano}?uf={UF}` | Marcar feriados nacionais e estaduais como contexto; não alterar compromisso nem vencimento automaticamente. |

Manter chamadas sob demanda, com timeout, limite de repetição e edição manual disponível em qualquer falha. Registrar BrasilAPI como origem e a data da consulta nos dados externos aceitos; guardar separado o que o usuário corrigiu. Seguir os termos e limites do serviço: sem crawling, varredura de faixas ou atualização em loop. Aplicar cache somente conforme a política de cada endpoint e nunca armazenar respostas de erro que a documentação proíba cachear. As respostas são referências para revisão, não confirmação de identidade, endereço atual, valor de venda ou presença física.

Referência: [documentação e termos da BrasilAPI](https://brasilapi.com.br/docs).

## Qualidade

Factories por estado de domínio e seeds com códigos estáveis, sem IDs mágicos de tags. Testes relevantes cobrem autorização cruzada, somas, concorrência, estornos, importação e falhas externas. Não criar testes que só repitam textos de documentação ou classes CSS.
