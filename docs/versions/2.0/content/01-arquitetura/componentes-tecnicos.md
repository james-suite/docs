---
id: componentes-tecnicos
title: Componentes técnicos planejados
description: Models, Actions, Policies, casts, Jobs e componentes de interface necessários por domínio.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: modelo-de-dados, migrations, enums, ui-ux-2-0, fases-2-0, assistente-pessoal-rag, migration-19-health, bibliotecas-candidatas, ecossistema-laravel
source_refs: https://brasilapi.com.br/docs, https://laravel.com/docs/13.x/helpers#numbers, https://carbon.nesbot.com/guide/getting-started/localization.html, https://laravel.com/docs/13.x/scout#meilisearch
updated: 2026-11-08
---

## Organização da implementação

CRUD simples segue o padrão vigente do projeto. Operações com várias entidades, concorrência, importação ou transporte externo justificam Actions/Services próprias. Não criar Repository, interface ou Service genérico para cada tabela. Interfaces são úteis nas fronteiras de provedor de IA, importador e transporte.

Priorizar as APIs nativas mapeadas em [Ecossistema Laravel](doc:ecossistema-laravel) para filas, agendamento, cache, HTTP, arquivos, validação e componentes Livewire. Os pacotes adicionais com benefício dependente de operação ou medição ficam separados para decisão do usuário nessa mesma página.

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

Os candidatos externos, licenças e pontos de integração estão em [Bibliotecas candidatas](doc:bibliotecas-candidatas). Essa seleção complementa as peças do James e do SGE inventariadas abaixo.

Reutilizar uma implementação quando o contrato, as dependências e a responsabilidade continuarem válidos. Portar uma peça não significa copiar o domínio ao redor. Conferir versões, licença, cobertura de testes e comportamento real antes de integrar no ciclo de desenvolvimento de 2027.

| Origem e referência | Caminho de reaproveitamento | Limites a respeitar |
| --- | --- | --- |
| SGE — `resources/views/components/select.blade.php`, `select-create.blade.php` | Portar o comportamento do combobox: teclado, busca, estados vazios e criação inline; adaptá-lo para TagPicker e seletores de Party. | A implementação atual integra Livewire 4, Flux 2 e Alpine. Rever versões no início do projeto. Usar Scout com Meilisearch nos seletores pesquisáveis; criar a tag continua sendo uma operação do domínio James. |
| SGE — `app/Support/SelectOptionSource.php`, `app/Concerns/SearchesSelectOptions.php` | Reaproveitar o contrato de opções limitadas, `hasMore` e busca com Scout/Meilisearch. | Adaptar campos, filtros e autorização ao espaço pessoal. A criação inline devolve a opção persistida diretamente ao seletor enquanto o índice é atualizado. |
| SGE — `app/Support/BrasilApiCepLookup.php`, `BrasilApiCompanyLookup.php`, `BrasilApiHolidayLookup.php` | Adaptar validação, timeout, normalização e retorno manual para os lookups de CEP, CNPJ e feriados do James. | Mapear os campos para Party/Place e registrar fonte/data quando aceitos. Não copiar DTOs do SGE nem confiar no resultado sem revisão. |
| SGE — `app/Helpers/BrazilianDocumentHelper.php`, `BrazilianContactHelper.php`, `BrazilianAddressHelper.php`, `DigitsHelper.php`, `CurrencyHelper.php`, `DateHelper.php` | Reaproveitar contratos úteis de normalização e apresentação, apoiando moeda/números em Number e datas em Carbon. | Concentrar locale, moeda e fuso no contexto compartilhado definido abaixo. Conferir precisão e formatos antes de portar cada função. |
| SGE — `app/Support/ActivityHistory.php`, `AdministrativeActivityPresenter.php`, `resources/views/components/audit/*` | Portar a separação entre consulta autorizada, apresentação e histórico contextual; incluir navegação e contexto geográfico próprios do James. | Não copiar escopos de campus, afiliação, estágio ou perfis acadêmicos. A autorização deve ser pelo usuário e Space do James. |
| James 1.x — `app/Services/Nfce/` (resolver, interface, scraper SVRS e DTOs) | Candidato principal a portar como adaptador fiscal determinístico para RS; mapear sua saída para FiscalDocument e FiscalDocumentExtraction. | Validar HTML/termos e fixtures por portal em 2027. O DTO atual não inclui filial nem código de produto. Preservar chave fiscal/idempotência e deixar IA apenas como fallback autorizado. |
| James 1.x — `app/Services/SettlementBalanceCalculator.php` | Reaproveitar cenários e a aritmética em centavos como referência para saldos por contraparte. | Recalcular a regra em cima de pagamentos e alocações 2.0; a API antiga depende de SettlementType e não deve virar contrato novo. |
| James 1.x — `app/Traits/HandlesAttachments.php`, `resources/views/components/media/manager.blade.php`, `AttachmentController.php` | Reaproveitar a experiência de enviar/listar/remover arquivos e servir download privado, refazendo a operação para criar Document/versão/vínculo. | Hoje os anexos pertencem diretamente a FinancialTransaction, Settlement ou SettlementGroup via Media Library; avatar de Contact é separado. No 2.0 o Media pertence à versão do Document, e todos os formulários reutilizam DocumentPicker/PrivateDocumentUpload. |
| James 1.x — `resources/views/components/form/*`, `components/ui/*`, `components/finance/*` | Usar casos e padrões visuais como referência; reconstruir componentes no stack escolhido para 2.0. | Não copiar em bloco: a seleção de tags atual presume catálogo já criado, vínculo pivô e fluxo de formulário próprios do 1.x. |
| James 1.x — `SettlementGroupService.php`, `FinancialTag.php` | Usar apenas para levantar casos e comportamento legado que a importação precisa reconhecer. | Não portar regras que reservam a tag `reembolso`, identificam tag por ID ou materializam “Minha Parte”. No 2.0, contraparte e pagador são relações explícitas; tag continua sendo classificação específica sem defaults. |

Extrair um pacote compartilhado para James e SGE só depois que ambos usarem o mesmo contrato estável. O candidato deve ficar independente de regras de domínio e ter versões, testes e responsabilidade de manutenção definidas. Até lá, portar e adaptar no destino evita dependência cruzada entre aplicações.

## Casts, valores e helpers

Money e Quantity precisam de operações exatas e arredondamento explícito. Avaliar `brick/money` e `brick/math` conforme [Bibliotecas candidatas](doc:bibliotecas-candidatas), com persistência e escalas do domínio. Campos de enum recebem cast próprio. Datas locais, instantes UTC e fuso IANA devem ser distinguidos no domínio. Conferir APIs e versões quando o desenvolvimento começar em 2027.

## Formatação e contexto regional compartilhados

Usar `Illuminate\Support\Number` para apresentação de números, percentuais, moedas e tamanhos de arquivo. Os helpers/componentes compartilhados resolvem locale e moeda pelo contexto, permitindo chamadas simples como `formatCurrency($valor)`. Quando o registro tem moeda própria, ela prevalece sobre o padrão do espaço. A conversão de unidades menores para apresentação conserva escala e precisão, inclusive para inteiros grandes. Aritmética monetária, rateios e arredondamento de negócio seguem o contrato de Money/Quantity. [Number do Laravel](https://laravel.com/docs/13.x/helpers#numbers).

Usar Carbon, já integrado ao Laravel, para datas, intervalos, comparações, cálculos temporais e apresentação localizada. Preferir `CarbonImmutable` nos cálculos, com política explícita de fim de mês para vencimentos e recorrências. Instantes são persistidos em UTC e convertidos ao fuso de apresentação com `setTimezone`, preservando o instante. Datas conhecidas somente por dia e eventos de dia inteiro conservam essa precisão. Configurar locale na instância/factory usada pelo contexto. [Localização do Carbon](https://carbon.nesbot.com/guide/getting-started/localization.html), [API do Carbon](https://carbon.nesbot.com/develop/reference.html).

Resolver o contexto regional uma vez na fronteira da requisição ou do job, junto ao PersonalSpaceContext, e reutilizá-lo em todos os módulos. A origem dos valores fica definida em um único contrato.

| Configuração | Fonte e precedência |
| --- | --- |
| Locale de apresentação | `user_preferences.locale`, com fallback validado na configuração da instalação. Normalizar o identificador conforme Number, Carbon e frontend na integração central. |
| Moeda padrão | `spaces.currency`, validada pelo catálogo ISO 4217. Serve como valor inicial de cadastros; registros existentes conservam a própria moeda. |
| Fuso padrão | `spaces.timezone`, validado como identificador IANA. Eventos e recorrências com fuso explícito conservam sua regra temporal. |
| Formatos, escalas e unidades | Catálogo e presets compartilhados para datas curtas/completas, quantidades, percentuais e moedas. Componentes recebem valores tipados e escolhem o preset adequado. |

Defaults da instalação ficam em `config/*`. Formulários, notificações, gráficos, PDFs e relatórios usam os mesmos formatadores. O frontend recebe uma representação normalizada desse contexto e os valores exatos, com formatação equivalente por Intl quando necessária. JSON de portabilidade mantém datas e números no formato canônico do schema, independente da apresentação localizada.

Preferências variáveis ficam isoladas por execução. Defaults estáticos de Number atendem à instalação, e contextos pessoais usam argumentos resolvidos pelo formatador ou escopo temporário restaurado. Jobs carregam identidade/escopo, resolvem suas preferências e limpam o contexto ao terminar, inclusive em falhas. Exportações reproduzíveis fixam as preferências no início e registram esse snapshot no manifesto. [Escopos de Number](https://laravel.com/docs/13.x/helpers#method-number-withlocale).

## Busca compartilhada com Scout e Meilisearch

Usar Laravel Scout com o driver Meilisearch para a busca textual e os seletores pesquisáveis dos módulos. Aproveitar a integração existente no SGE, adaptando documentos indexáveis, campos pesquisáveis, filtros e ordenação ao domínio James. O PostgreSQL mantém os fatos, relações, cálculos e validações transacionais. [Integração Scout/Meilisearch](https://laravel.com/docs/13.x/scout#meilisearch).

As consultas passam pelo backend Laravel, que impõe `space_id` antes da busca e configura esse atributo como filtrável. Respostas, contagens, facetas e sugestões respeitam esse filtro. Antes de exibir registros ou executar ações, revalidar no banco o proprietário, a Policy e o estado atual. Indexar somente os campos necessários, conforme a política do módulo. Diário, documentos e saúde seguem os controles de inclusão definidos no domínio. [Isolamento de buscas no Meilisearch](https://www.meilisearch.com/blog/multi-tenancy).

Sincronizar o índice em fila após commit. Jobs recarregam o estado atual, tratam exclusão/revogação e permitem reconstrução a partir dos fatos autorizados. A indexação assíncrona exige feedback de atualização e entrega direta do registro recém-criado ao seletor. Configurar índices por ambiente, campos filtráveis/ordenáveis, retentativas e observação das falhas. [Filas e atualização do Scout](https://laravel.com/docs/13.x/scout#queueing).

O fluxo de restauração reconstrói o índice e valida seu isolamento antes de reabrir a busca. A evolução semântica e os embeddings seguem a fase de [RAG](doc:assistente-pessoal-rag), com avaliação própria do backend vetorial.

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

## Componentes compartilhados de seleção e análise

PartyPicker oferece seleção única/múltipla, papéis, criação pendente e desambiguação. SplitEditor mantém valores por identidade e SettlementPaymentEditor mantém as alocações por obrigação. A consulta de opções é limitada e autorizada. Reaproveitar do SGE o comportamento de combobox e as fontes de opções já inventariadas, adaptando nomes, tipos e cardinalidades.

DimensionFilter apresenta chips tipados de Category, Tag, Party, Vehicle, Product e Place. O serviço de relatório resolve cada dimensão em sua relação de domínio e expõe medida, período, moeda, cobertura e sobreposição. FiscalAdjustmentEditor e AllocateTransactionAdjustment compartilham o contrato de valores exatos e revisão de alocações.

ExportAccountData, ExportFilteredReport e o catálogo de exportadores por domínio devem reutilizar normalizadores de dinheiro, datas, dimensões e IDs estáveis. Cada resultado declara schema e versão. O backup integral da instalação usa o procedimento operacional documentado em [Portabilidade e backup](doc:portabilidade-backup).

## Contratos da revisão integrada

DocumentVersion recebe publicação idempotente e vínculo de evidência fixo. AllocateSplitFunding e ReverseSettlement coordenam financiamento parcial e correção de acertos com o financeiro. Revisões de preço conservam uma única observação corrente. As responsabilidades e a ordem de correção estão em [Integrações e consistência](doc:integracoes-e-consistencia).

Uma infraestrutura de chamada externa pode compartilhar validação de destinos, limites e diagnóstico. Cada adaptador mantém seu parser, campos autorizados e política de revisão. Exportações e importações usam snapshots, lease recuperável e retenção temporária dos arquivos referenciados.
