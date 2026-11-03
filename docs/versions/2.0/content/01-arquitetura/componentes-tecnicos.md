---
id: componentes-tecnicos
title: Componentes técnicos planejados
description: Models, Actions, Policies, casts, Jobs e componentes de interface necessários por domínio.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: modelo-de-dados, migrations, enums, ui-ux-2-0, fases-2-0, assistente-pessoal-rag, migration-19-health
source_refs: https://brasilapi.com.br/docs
updated: 2026-11-03
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
| Acertos | SettlementGroup, SplitParticipant, SplitShare, SplitContribution, SettlementObligation, SettlementPayment, SettlementPaymentAllocation | Rateio sem parcela própria, contribuições sem duplicar a saída original, liquidação parcial ligada a pagamentos financeiros opcionais e projeções por contraparte. |
| Notificações | Notification, NotificationPreference, NotificationEndpoint, NotificationDelivery, NotificationDeliveryAttempt | Outbox, digest, reenvio, resolução/adiamento e deduplicação durável. |
| IA | AiConnection, AiRun, ClassificationSuggestion, CalendarCommandProposal, AiUsageEntry, AiBudgetPeriod | Seleção de conexão autorizada, reserva de orçamento, classificação/aplicação de tags, comandos de agenda e captura tipada de transação por texto, se aprovados. |
| Assistente pessoal (futuro) | AiConversation, AiMessage, AiMemorySource, AiMemoryChunk, AssistantActionProposal | Busca híbrida com filtros por espaço/fonte, respostas com referências, consulta SQL de domínio para números, indexação idempotente e propostas por ferramentas allowlisted; extensão separada, não necessária ao primeiro lançamento. |
| Captura/diário | Place, PlaceIdentifier, Visit, JournalEntry, JournalEntryRevision, ActivityLocationContext | Filial/endereço, identificadores empresariais, check-in pelo site, contexto geográfico por ação auditada, diário versionado e vínculos. |
| Veículos | Vehicle, VehicleEvent, VehicleEventItem, FuelEvent, VehicleFinancialLink, OdometerReading, MaintenancePlan | Lançamento independente de odômetro, leitura de nota, resolução do posto/filial, vínculo sem duplicar custos, diário genérico, serviços reais com vários itens, consumo, correção de odômetro e vencimento por km/data. |
| Organização | Task, Checklist, ChecklistItem, CalendarEvent, CalendarSeries, CalendarFeedToken, CalendarConnection, CalendarEventExternalLink | Tarefas, checklist histórico, exceções de série, lembretes, feed privado `.ics` e conexão bidirecional opcional posterior via CalDAV/Google. |
| Patrimônio/saúde | Asset, AssetValuation, Exercise, WorkoutRoutine, WorkoutSession, WorkoutSet, HealthObservation, HealthImportBatch (condicional) | Evolução patrimonial sem duplicação; treino real e unidades; avaliar importação de medidas e treinos por XML enviado no navegador, com proveniência, idempotência e revisão. Séries de atividade e rotas só entram se o formato real as trouxer e houver uso. |
| Documentos | Document, DocumentVersion sobre Media Library, RecordLink | Uploads contextuais e biblioteca usam os mesmos objetos privados. Um documento pode ficar sem vínculo ou ser relacionado a vários Records; versão imutável, checksum para aviso de duplicata e prazos sem estado duplicado. |
| Dados públicos do Brasil | BrasilApiReferenceLookup | Consultas sob demanda de CNPJ, CEP, bancos, NCM, FIPE e feriados; registrar origem e data de consulta quando a resposta alimentar um cadastro. |
| Portabilidade | DataExport no 1.x; ImportRun, ImportItem, LegacyIdMap no 2.0 | Gerar arquivo, analisar upload, prévia, confirmar na própria conta, reconciliar e limpar staging. |

## Reaproveitamento de código

Reutilizar uma implementação quando o contrato, as dependências e a responsabilidade continuarem válidos. Portar uma peça não significa copiar o domínio ao redor. Conferir versões, licença, cobertura de testes e comportamento real antes de integrar no ciclo de desenvolvimento de 2027.

| Origem e referência | Caminho de reaproveitamento | Limites a respeitar |
| --- | --- | --- |
| SGE — `resources/views/components/select.blade.php`, `select-create.blade.php` | Portar o comportamento do combobox: teclado, busca, estados vazios e criação inline; adaptá-lo para TagPicker e seletores de Party. | A implementação atual integra Livewire 4, Flux 2 e Alpine. Rever versões no início do projeto; não carregar busca remota e Meilisearch se os catálogos não precisarem disso. Criar a tag continua sendo uma operação do domínio James. |
| SGE — `app/Support/SelectOptionSource.php`, `app/Concerns/SearchesSelectOptions.php` | Reaproveitar o contrato de opções limitadas, `hasMore` e busca remota como referência. | As classes incorporam Scout/Meilisearch e consultas acadêmicas. Começar com Eloquent; adotar busca remota quando volume e medição justificarem. |
| SGE — `app/Support/BrasilApiCepLookup.php`, `BrasilApiCompanyLookup.php`, `BrasilApiHolidayLookup.php` | Adaptar validação, timeout, normalização e retorno manual para os lookups de CEP, CNPJ e feriados do James. | Mapear os campos para Party/Place e registrar fonte/data quando aceitos. Não copiar DTOs do SGE nem confiar no resultado sem revisão. |
| SGE — `app/Helpers/BrazilianDocumentHelper.php`, `BrazilianContactHelper.php`, `BrazilianAddressHelper.php`, `DigitsHelper.php`, `CurrencyHelper.php`, `DateHelper.php` | Reaproveitar ou portar funções pontuais de normalização e apresentação que tenham a mesma semântica no James. | Conferir formatos e testes antes de copiar. Formatação monetária não substitui aritmética exata em unidades menores; datas precisam respeitar o fuso do espaço. |
| SGE — `app/Support/ActivityHistory.php`, `AdministrativeActivityPresenter.php`, `resources/views/components/audit/*` | Portar a separação entre consulta autorizada, apresentação e histórico contextual; incluir navegação e contexto geográfico próprios do James. | Não copiar escopos de campus, afiliação, estágio ou perfis acadêmicos. A autorização deve ser pelo usuário e Space do James. |
| James 1.x — `app/Services/Nfce/` (resolver, interface, scraper SVRS e DTOs) | Candidato principal a portar como adaptador fiscal determinístico para RS; mapear sua saída para FiscalDocument e FiscalDocumentExtraction. | Validar HTML/termos e fixtures por portal em 2027. O DTO atual não inclui filial nem código de produto. Preservar chave fiscal/idempotência e deixar IA apenas como fallback autorizado. |
| James 1.x — `app/Services/SettlementBalanceCalculator.php` | Reaproveitar cenários e a aritmética em centavos como referência para saldos por contraparte. | Recalcular a regra em cima de pagamentos e alocações 2.0; a API antiga depende de SettlementType e não deve virar contrato novo. |
| James 1.x — `app/Traits/HandlesAttachments.php`, `resources/views/components/media/manager.blade.php`, `AttachmentController.php` | Reaproveitar a experiência de enviar/listar/remover arquivos e servir download privado, refazendo a operação para criar Document/versão/vínculo. | Hoje os anexos pertencem diretamente a FinancialTransaction, Settlement ou SettlementGroup via Media Library; avatar de Contact é separado. No 2.0 o Media pertence à versão do Document, e todos os formulários reutilizam DocumentPicker/PrivateDocumentUpload. |
| James 1.x — `resources/views/components/form/*`, `components/ui/*`, `components/finance/*` | Usar casos e padrões visuais como referência; reconstruir componentes no stack escolhido para 2.0. | Não copiar em bloco: a seleção de tags atual presume catálogo já criado, vínculo pivô e fluxo de formulário próprios do 1.x. |
| James 1.x — `SettlementGroupService.php`, `FinancialTag.php` | Usar apenas para levantar casos e comportamento legado que a importação precisa reconhecer. | Não portar regras que reservam a tag `reembolso`, identificam tag por ID ou materializam “Minha Parte”. No 2.0, contraparte e pagador são relações explícitas; tag continua sendo classificação específica sem defaults. |

Extrair um pacote compartilhado para James e SGE só depois que ambos usarem o mesmo contrato estável. O candidato deve ficar independente de regras de domínio e ter versões, testes e responsabilidade de manutenção definidas. Até lá, portar e adaptar no destino evita dependência cruzada entre aplicações.

## Casts, valores e helpers

Money e Quantity precisam de operações exatas e arredondamento explícito; usar biblioteca compatível ou objetos pequenos em vez de espalhar casts para float. Campos de enum recebem cast próprio. Datas locais, instantes UTC e fuso IANA devem ser distinguidos no domínio. Conferir APIs e versões dos candidatos listados acima quando o desenvolvimento começar em 2027.

## Policies e validação

Cada recurso tem Policy por proprietário, além das verificações de estado. Seletores, relacionamentos, anexos, revisões, busca, relatórios, auditoria e downloads recebem o mesmo escopo. Administrador da instalação administra contas e operação; isso não concede leitura de diário ou finanças dos usuários na interface.

Form Requests/validação Livewire aceitam somente dados do formulário; space_id, autoria e idempotência são resolvidos no servidor. Revalidar dentro da transação os registros que podem ter sido arquivados ou alterados depois de o formulário abrir.

## Jobs e processos agendados

Gerar exportação, processar em filas importações privadas que forem aprovadas (migração James 1.x e XML de saúde), enviar notificações, materializar recorrências, sincronizar calendários externos e chamar IA. Arquivos enviados ao servidor passam por staging privado, limite de tamanho, prévia, idempotência e limpeza. Nenhum worker depende de app móvel ou acesso contínuo a serviço externo. Job carrega IDs de domínio, nunca senha temporária ou API key serializada. Definir timeout, backoff, erro sanitizado e recuperação após interrupção. UNIQUE no banco continua sendo a garantia final de idempotência.

## Componentes de UI

TagPicker com criação inline a partir de `select-create` do SGE e explicação da origem da tag; FiscalDocumentCapture com revisão campo a campo; PaymentEditor para vários meios/datas e pagadores; SplitEditor com lista explícita de consumidores/pagadores; SettlementPaymentEditor para liquidar obrigações sem registrar outra compra; PartySettlementSummary para separar consumo, adiantamentos, saldo aberto e caixa por contato/empresa; InvoicePaymentsHistory; NotificationInbox; ImportWizard; VehicleFinancialLinkPicker com exibição como chip contextual/filtro; VehicleServiceEditor com vários itens e data real; VehicleDiaryEditor; PrivateFileUpload; AuditHistory.

Incluir também AiTagClassifier com criação/deduplicação de tag específica autorizada, FiscalIssuerPlaceMatcher, AiTransactionComposer, ProductMatcher com identificação por empresa/estabelecimento e criação revisável, ProductListingReview, VariableMeasureCodeReview com peso/preço/total e origem visíveis, PriceHistory, PersonalPriceBasket, BrowserLocationCapture no salvamento, ActivityLocationMap, OdometerQuickEntry independente do abastecimento e CalendarCommandComposer com revisão de alvo/alterações. Para documentos, PrivateDocumentUpload, DocumentPicker, DocumentLibrarySearch, DocumentPreview, DocumentVersionHistory e DocumentExpiryEditor; todos os módulos chamam o mesmo fluxo de envio/vínculo e autorização do documento. Preview e download do original aplicam a mesma Policy. Para saúde, HealthExportUpload, HealthExportReview e, se o arquivo validado trouxer séries, WorkoutHeartRateChart; não planejar conexão HealthKit. Para a evolução futura: AssistantConversation com fontes clicáveis, AnswerEvidenceList e AssistantActionReview; não ocultar de onde vieram as respostas nem aplicar proposta sem autorização do domínio. A captura geográfica deve associar-se por correlação ao evento de auditoria; o mapa usa tiles de provedor configurável e respeita retenção da localização. O decoder de etiqueta deve ser determinístico; só perfis testados com etiquetas reais podem preencher candidatos. Os nomes são propostas de responsabilidades, não classes já criadas. Cada componente deve ter loading, validação, teclado e retorno ao contexto anterior. Flux 2 gratuito fornece a base; componentes próprios cobrem o domínio sem depender de recursos Pro.

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
