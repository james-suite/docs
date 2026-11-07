---
id: migration-06-transactions
title: Migration 06 — Transações, itens e origem fiscal
description: Contrato lógico de tabelas, FKs, índices e conversão: transações, itens e leitura de documentos fiscais.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, financas, ia-e-memoria, produtos-precos, veiculos, documentos-arquivos, migration-15-vehicles, migration-21-products, migration-22-documents, componentes-tecnicos, enum-fiscal-capture-method, enum-fiscal-extraction-status
source_refs: https://brasilapi.com.br/docs
updated: 2026-11-07
---

## Leitor legado e cobertura

A versão 1.x já contém NfceSourceResolver e SvrsNfceScraper, incluindo a rota de redirecionamento do portal RS para SVRS. O DTO atual carrega emitente/documento, data, total e descrição/quantidade/preço dos itens; não carrega endereço do estabelecimento nem código do produto. O mapa de prefixos de UF identifica a UF da chave, mas não comprova cobertura de um scraper para cada portal. Ao portar, verificar fixtures por origem e ampliar campos de forma explícita; não reutilizar cálculos em float nem marcar como suportada uma UF sem validação.

## `financial_transactions`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, kind, purpose, state` | Registro e enums TransactionKind, TransactionPurpose e TransactionState. Entrada por texto com IA usa Draft até confirmação financeira. |
| `description, occurred_on, due_on` | Descrição, data econômica e vencimento opcional. |
| `amount_minor, currency` | Valor total positivo e moeda. |
| `party_id, default_category_id, ownership` | Contraparte, categoria de fallback e ItemOwnership para consumo do cabeçalho. Cotas de rateio confirmadas são a fonte do valor por participante. |
| `related_transaction_id` | Self-FK opcional para devolução/ajuste; validar propósito e espaço. |
| `notes_markdown, legacy_metadata` | Observações e metadados legados versionados, sem substituir campos tipados. |

## `financial_transaction_items`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, transaction_id, position` | Identidade vinculável, pai do mesmo espaço e ordem única por transação. |
| `description, quantity, quantity_unit_label, unit_price` | Quantidade e preço exatos com escala e unidade informada na linha; guardar o rótulo original da unidade. Não usar float. |
| `fiscal_line_snapshot, fiscal_line_schema_version` | Snapshot limitado dos campos fiscais originais, com identificação da linha e origem. Revisões de valores confirmados conservam esse snapshot e a versão do arquivo/payload. |
| `fiscal_ncm_code` | Código NCM opcional copiado da linha fiscal original; consulta externa pode validar descrição e vigência, sem transformar NCM em identidade de produto ou tag. |
| `gross_minor, discount_minor, surcharge_minor, total_minor` | Valores originais da linha. total_minor = gross − discount + surcharge. Ajustes globais atribuídos ficam nas alocações e formam o líquido analítico da linha. |
| `is_legacy_summary` | Item sintético de resumo quando o 1.x não tinha itens; quantity/unit_price podem ser nulos nesse caso, sem inventar produtos ou quantidades. |
| `category_id, ownership` | Categoria única e ItemOwnership para consumo próprio/terceiros. |

Itens financeiros são opcionais. Uma transação simples sem nota pode guardar descrição, data e total sem inventar produto ou resumo sintético; quando houver itens, os totais e ajustes globais precisam reconciliar com o cabeçalho. `is_legacy_summary` permanece reservado a linhas sintéticas de importação do 1.x.

Tags são atribuídas ao `record_id` da transação ou de cada item por `tag_assignments`. Preservar esses dois níveis de forma independente. Uma transação sem itens continua tendo tags de cabeçalho; em compras detalhadas, tags de cabeçalho dão contexto geral e tags de item classificam linhas específicas. A busca pode consultar ambas, mas não replicar assignments nem contar o total da transação uma vez por item.

## `fiscal_documents`

| Campo ou grupo | Contrato |
| --- | --- |
| `transaction_id, document_kind, access_key` | Documento de origem; chave única por espaço quando informada. |
| `issuer_party_id, issuer_place_id, issued_at, source_url` | Empresa emitente, estabelecimento opcional e procedência; place FK nullable acrescentada após migration 13. URL não deve ser buscada por job sem validação de host. |
| `capture_method, extraction_status, extractor_version` | Método de entrada e estado/versão do processo de leitura; consultar enums próprios. |
| `extracted_at, reviewed_at` | Instantes da extração e da confirmação humana, separados da data fiscal. |
| `source_payload_version, source_document_version_id` | Metadados estruturados e FK opcional para document_versions.id, fixando os bytes usados na leitura. Um PDF pode conter mais de uma nota, com identidades fiscais distintas. |

## `fiscal_document_extractions`

| Campo ou grupo | Contrato |
| --- | --- |
| `fiscal_document_id, revision, status, source_document_version_id, input_hash, source_revision` | Uma tentativa/revisão de leitura com versão/hash da entrada e revisão esperada. Referência ao arquivo é opcional quando a origem é payload fiscal estruturado. |
| `candidate_data, confidence_by_field` | Campos sugeridos e confiança por campo para revisão; privados e limitados ao documento. |
| `method, processor_code, processor_version, jurisdiction_code, completed_at, reviewed_at` | Tipo/nome/versionamento do leitor e UF aplicável; identifica scraper determinístico, OCR ou provedor de IA sem enum frágil de integrações. |

Após aceitar a leitura, os valores confirmados ficam em `financial_transactions` e `financial_transaction_items`. Uma revisão nova não sobrescreve silenciosamente os dados já confirmados; a pessoa corrige o rascunho ou cria uma revisão explícita.

## Integridade, índices e comportamento

- Registrar transação, itens e ajustes atomicamente. O total concilia líquidos das linhas e ajustes globais, inclusive resíduos sem atribuição. Atribuir parcialmente desconto global é permitido. Preservar centavos, procedência e revisão de cada alocação.
- Leitura de XML/QR/imagem produz candidatos, não lançamento confirmado. Validar duplicidade por chave fiscal e, quando ausente, por impressão digital; preservar o original em armazenamento privado e não buscar URL arbitrária extraída do QR.
- Uma transação digitada ou montada por `TransactionDraft` não precisa de `fiscal_documents`. Criar como Draft e confirmar só após revisão; não criar item, categoria ou pagamento ausente no texto.
- Captura deve preencher emitente, estabelecimento, data e linhas quando reconhecíveis e mostrar confiança/itens ambíguos para revisão. Um nome de rede não identifica sozinho a filial. Pagador e conta não são presumidos a partir da nota fiscal.
- Ordem de leitura: payload/XML fiscal disponível, adaptador determinístico compatível com a UF e, somente se permitido, tarefa de IA FiscalExtraction. Todo resultado permanece em rascunho para conferência; armazenar código/versão do processador, UF, custo quando houver e confiança por campo.
- A tarefa de IA usa o pipeline autorizado do pacote 12, seu limite de gasto e a permissão específica de conteúdo fiscal. Recusa, indisponibilidade ou falha não bloqueia revisão e entrada manual; não enviar a nota a outro provedor como fallback invisível.
- Quando a linha da nota contiver NCM, permitir consultar o catálogo da BrasilAPI para mostrar descrição e vigência do código na data fiscal. Preservar o código original; não inferir NCM pelo nome do produto nem usar a descrição fiscal para mesclar produtos ou atribuir tags.
- Linhas fiscais podem receber product_id, product_listing_id e base_quantity/base_unit por FKs adicionadas no pacote 21. A observação de preço só nasce após confirmar a identidade do produto, a compra e a quantidade/unidade comparável. Se uma etiqueta/código de balança também for lido, associá-lo à linha fiscal correspondente; não criar uma segunda compra nem uma segunda observação para o mesmo item.
- A atribuição automática integral do desconto global exige uma nota com um único item elegível e quantidade conhecida. Em nota com vários itens, a pessoa pode atribuir qualquer parte confirmada do desconto a uma ou mais linhas e manter saldo global residual conciliado.
- Sem itens no legado, criar um resumo identificado com valor conhecido, sem fabricar produto ou quantidade. Substituir o resumo por itens detalhados exige reconciliação e histórico.
- Vínculos a veículos usam `vehicle_financial_links` no pacote 15: transação inteira para despesa integralmente destinada ao veículo, item quando apenas aquela linha da compra pertence ao veículo. Não gravar `vehicle_id` ou tag sintética no núcleo financeiro.
- Cancelar documento confirmado exige nenhum pagamento efetivo. Devolução comercial preserva o documento original e cria outro relacionado; reversão técnica é uma operação diferente.
- Não armazenar financial_account_id como único destino da transação: formas de pagamento passam pelo pacote 07.
- Estado de quitação é projeção de pagamentos válidos. Estorno de uma compra cria documento de devolução relacionado e operação inversa; não reabrir cobrança como se fosse falta de pagamento.
- UNIQUE(space_id, id), índice por espaço/data e por contraparte/data; indexar campos de busca com dados e volume representativos.

## Ajustes globais e suas alocações

| Tabela | Campos e contrato |
| --- | --- |
| financial_transaction_adjustments | id, space_id, record_id único, transaction_id, category_id opcional, direction (Discount ou Surcharge), amount_minor positivo, description, source_reference, revision. Cada ajuste global existe uma vez e conserva a origem fiscal/manual. |
| financial_adjustment_allocations | space_id, adjustment_id, transaction_item_id, amount_minor positivo, origin, reason. UNIQUE(adjustment_id, transaction_item_id). Item e ajuste devem pertencer à mesma transação/espaço. |

A direção do ajuste usa um enum fechado FinancialAdjustmentDirection. O campo origin identifica leitor determinístico ou revisão manual. A classificação inicial por IA mantém a revisão fiscal antes de confirmar candidatos.

Líquido analítico do item = total_minor original − descontos globais alocados + acréscimos globais alocados. A transação totaliza esse líquido e o resíduo global. O banco e a operação devem garantir que as alocações somadas nunca superem cada ajuste, validando sob bloqueio transacional. Desconto aplicado ao item respeita seu valor elegível. Recalcular preços normalizados em decimal exato.

A interface pode confirmar o documento financeiro com resíduo global identificado. O preço comparável conserva status de cobertura pendente enquanto houver ajuste residual potencialmente aplicável àquela linha. O resumo de categoria mostra o resíduo como parcela de ajuste ainda sem classificação, permitindo reconciliar o relatório com o total financeiro.

Adicionar metadados originais da nota para distinguir desconto já discriminado em cada linha de desconto realmente adicional no cabeçalho. Quando o total global apenas resume descontos das linhas, marcar como resumo informativo e evitar criar outro ajuste financeiro.

## Compatibilidade de linhas negativas e classificação dos ajustes

Linhas negativas do 1.x são convertidas em FinancialAdjustment com valor positivo e direção explícita, record_id próprio e snapshot assinado da linha original. TagAssignments e eventual categoria permanecem vinculados ao Record do ajuste. A chave de origem distingue esse papel da conversão de um item usual.

O relatório considera o ajuste uma única vez, com sinal de sua direção. Atribuir seu efeito a itens conserva a identidade e a classificação do ajuste. Filtros de tags sobre o ajuste consultam seu valor assinado, e tags do item consultam o líquido do item com a política de sobreposição declarada.

## Revisão de valores já relacionados

Reclassificar tags/categorias ou atribuir desconto global sem alterar o total usa revisão explícita e revalida preços e vínculos afetados. Alterar total/moeda ou remover item com pagamentos, rateio, devolução ou alocação de veículo exige plano de correção antes da gravação. O comando financeiro expõe as dependências e preserva os lançamentos efetivados.

A interpretação fiscal mantém bytes/payload de origem, valores lidos e valores confirmados com revisões distinguíveis. Aceitar outra extração de nota já confirmada apresenta diferenças e usa a mesma operação de revisão.

## Conversão do James 1.x

Preservar amount, description, date, status, itens e NFC-e. Valores decimais convertidos por texto para unidades mínimas. Migrar tags do cabeçalho e dos itens como vínculos separados; não inferir tags dos itens a partir do cabeçalho. O arquivo de nota vai para `Document`/versão do pacote 22; `FiscalDocument` referencia esse mesmo documento. O conversor distingue documento fiscal já importado de captura por imagem e nunca inventa pagador, desconto por item, produto de catálogo ou veículo. Emitente e filial importados são candidatos até uma correspondência segura.

## Componentes previstos

SaveTransaction; ConfirmTransaction; CancelTransaction; ImportFiscalDocument; ExtractFiscalDocument; ReviewFiscalExtraction; casts Money/Quantity; query de relatórios por base declarada.
