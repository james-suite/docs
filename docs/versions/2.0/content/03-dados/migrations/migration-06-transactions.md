---
id: migration-06-transactions
title: Migration 06 — Transações, itens e origem fiscal
description: Contrato lógico de tabelas, FKs, índices e conversão: transações, itens e leitura de documentos fiscais.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, financas, ia-e-memoria, produtos-precos, veiculos, migration-15-vehicles, migration-21-products, componentes-tecnicos, enum-fiscal-capture-method, enum-fiscal-extraction-status
source_refs: https://brasilapi.com.br/docs
updated: 2026-10-29
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 03, 04. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

## Leitor legado e cobertura

A versão 1.x já contém NfceSourceResolver e SvrsNfceScraper, incluindo a rota de redirecionamento do portal RS para SVRS. O DTO atual carrega emitente/documento, data, total e descrição/quantidade/preço dos itens; não carrega endereço do estabelecimento nem código do produto. O mapa de prefixos de UF identifica a UF da chave, mas não comprova cobertura de um scraper para cada portal. Ao portar, verificar fixtures por origem e ampliar campos de forma explícita; não reutilizar cálculos em float nem marcar como suportada uma UF sem validação.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `financial_transactions`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, kind, purpose, state` | Registro e enums TransactionKind, TransactionPurpose e TransactionState. Entrada por texto com IA usa Draft até confirmação financeira. |
| `description, occurred_on, due_on` | Descrição, data econômica e vencimento opcional. |
| `amount_minor, currency` | Valor total positivo e moeda. |
| `party_id, default_category_id` | Contraparte e classificação de fallback opcionais. |
| `related_transaction_id` | Self-FK opcional para devolução/ajuste; validar propósito e espaço. |
| `notes_markdown, legacy_metadata` | Observações e metadados legados versionados, sem substituir campos tipados. |

## `financial_transaction_items`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, transaction_id, position` | Identidade vinculável, pai do mesmo espaço e ordem única por transação. |
| `description, quantity, unit_price` | Quantidade e preço exatos com escala; não float. |
| `fiscal_ncm_code` | Código NCM opcional copiado da linha fiscal original; consulta externa pode validar descrição e vigência, sem transformar NCM em identidade de produto ou tag. |
| `gross_minor, discount_minor, surcharge_minor, total_minor` | gross − discount + surcharge = total; soma dos totais dos itens igual ao cabeçalho. |
| `is_legacy_summary` | Item sintético de resumo quando o 1.x não tinha itens; quantity/unit_price podem ser nulos nesse caso, sem inventar produtos ou quantidades. |
| `category_id, ownership` | Categoria única e ItemOwnership para consumo próprio/terceiros. |

Itens financeiros são opcionais. Uma transação simples sem nota pode guardar descrição, data e total sem inventar produto ou resumo sintético; quando houver itens, os totais precisam reconciliar com o cabeçalho. `is_legacy_summary` permanece reservado a linhas sintéticas de importação do 1.x.

## `fiscal_documents`

| Campo ou grupo | Contrato |
| --- | --- |
| `transaction_id, document_kind, access_key` | Documento de origem; chave única por espaço quando informada. |
| `issuer_party_id, issuer_place_id, issued_at, source_url` | Empresa emitente, estabelecimento opcional e procedência; place FK nullable acrescentada após migration 13. URL não deve ser buscada por job sem validação de host. |
| `capture_method, extraction_status, extractor_version` | Método de entrada e estado/versão do processo de leitura; consultar enums próprios. |
| `extracted_at, reviewed_at` | Instantes da extração e da confirmação humana, separados da data fiscal. |
| `source_payload_version, raw_file_media_id` | Metadados estruturados e arquivo privado opcional. |

## `fiscal_document_extractions`

| Campo ou grupo | Contrato |
| --- | --- |
| `fiscal_document_id, revision, status` | Uma tentativa/revisão de leitura; não substitui os campos tipados da transação. |
| `candidate_data, confidence_by_field` | Campos sugeridos e confiança por campo para revisão; privados e limitados ao documento. |
| `method, processor_code, processor_version, jurisdiction_code, completed_at, reviewed_at` | Tipo/nome/versionamento do leitor e UF aplicável; identifica scraper determinístico, OCR ou provedor de IA sem enum frágil de integrações. |

Após aceitar a leitura, os valores confirmados ficam em `financial_transactions` e `financial_transaction_items`. Uma revisão nova não sobrescreve silenciosamente os dados já confirmados; a pessoa corrige o rascunho ou cria uma revisão explícita.

## Integridade, índices e comportamento

- Registrar transação e itens atomicamente; total consistente e escala de moeda explícita. Desconto/acréscimo do cabeçalho deve ser distribuído nos itens por regra determinística ou representado por item de ajuste; não contar o mesmo ajuste duas vezes.
- Leitura de XML/QR/imagem produz candidatos, não lançamento confirmado. Validar duplicidade por chave fiscal e, quando ausente, por impressão digital; preservar o original em armazenamento privado e não buscar URL arbitrária extraída do QR.
- Uma transação digitada ou montada por `TransactionDraft` não precisa de `fiscal_documents`. Criar como Draft e confirmar só após revisão; não criar item, categoria ou pagamento ausente no texto.
- Captura deve preencher emitente, estabelecimento, data e linhas quando reconhecíveis e mostrar confiança/itens ambíguos para revisão. Um nome de rede não identifica sozinho a filial. Pagador e conta não são presumidos a partir da nota fiscal.
- Ordem de leitura: payload/XML fiscal disponível, adaptador determinístico compatível com a UF e, somente se permitido, tarefa de IA FiscalExtraction. Todo resultado permanece em rascunho para conferência; armazenar código/versão do processador, UF, custo quando houver e confiança por campo.
- A tarefa de IA usa o pipeline autorizado do pacote 12, seu limite de gasto e a permissão específica de conteúdo fiscal. Recusa, indisponibilidade ou falha não bloqueia revisão e entrada manual; não enviar a nota a outro provedor como fallback invisível.
- Quando a linha da nota contiver NCM, permitir consultar o catálogo da BrasilAPI para mostrar descrição e vigência do código na data fiscal. Preservar o código original; não inferir NCM pelo nome do produto nem usar a descrição fiscal para mesclar produtos ou atribuir tags.
- Linhas fiscais podem receber product_id e base_quantity/base_unit por FKs adicionadas no pacote 21. A observação de preço só nasce após confirmar a identidade do produto, a compra e a quantidade/unidade comparável.
- Só aplicar desconto geral ao preço unitário de combustível automaticamente quando há um único item de combustível e quantidade identificável. Nota com vários itens exige atribuição explícita do desconto; valor total e linhas precisam reconciliar antes de confirmar.
- Sem itens no legado, criar um resumo identificado com valor conhecido, sem fabricar produto ou quantidade. Substituir o resumo por itens detalhados exige reconciliação e histórico.
- Cancelar documento confirmado exige nenhum pagamento efetivo. Devolução comercial preserva o documento original e cria outro relacionado; reversão técnica é uma operação diferente.
- Não armazenar financial_account_id como único destino da transação: formas de pagamento passam pelo pacote 07.
- Estado de quitação é projeção de pagamentos válidos. Estorno de uma compra cria documento de devolução relacionado e operação inversa; não reabrir cobrança como se fosse falta de pagamento.
- UNIQUE(space_id, id), índice por espaço/data e por contraparte/data; indexar campos de busca com dados e volume representativos.

## Conversão do James 1.x

Preservar amount, description, date, status, itens e NFC-e. Valores decimais convertidos por texto para unidades mínimas. Datas e tags dos itens não são inferidas a partir do cabeçalho quando já existem. O conversor distingue documento fiscal já importado de captura por imagem e nunca inventa pagador, desconto por item, produto de catálogo ou veículo. Emitente e filial importados são candidatos até uma correspondência segura.

## Componentes previstos

SaveTransaction; ConfirmTransaction; CancelTransaction; ImportFiscalDocument; ExtractFiscalDocument; ReviewFiscalExtraction; casts Money/Quantity; query de relatórios por base declarada.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
