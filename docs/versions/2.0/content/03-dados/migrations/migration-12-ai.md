---
id: migration-12-ai
title: Migration 12 — IA, credenciais e sugestões
description: Contrato lógico de tabelas, FKs, índices e conversão: IA, credenciais e propostas.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, ia-e-memoria, assistente-pessoal-rag, agenda-tarefas, financas, componentes-tecnicos
source_refs: https://platform.openai.com/docs/guides/embeddings, https://github.com/pgvector/pgvector
updated: 2026-10-29
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 04, 06. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `ai_connections`

| Campo ou grupo | Contrato |
| --- | --- |
| `scope, owner_user_id, provider_code, model` | AiConnectionScope; dono obrigatório somente em conexão pessoal. |
| `secret_ciphertext ou secret_reference` | Segredo criptografado pessoal ou referência ao cofre/ambiente da instalação; nunca ambos. |
| `disabled_at, verified_at` | Situação operacional sem expor a chave. |

## `ai_user_settings`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id, connection_id, enabled, classification_mode, policy_revision` | Seleção explícita de conexão; ClassificationMode com padrão Suggest e revisão da preferência para autoria da automação. |
| `feature_permissions, budget_limit_minor, budget_currency` | Consentimentos separados para `tag_suggestion`, `fiscal_extraction` e, se aprovadas, `calendar_command` e `transaction_draft`; orçamento aplicável por função/conexão. Futuramente, `assistant_answer`, `memory_indexing` e `assistant_action` recebem consentimentos separados. Cada função começa desativada até autorização explícita. |

## `ai_runs`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id, space_id, connection_id, task, status` | Solicitante e contexto imutáveis. |
| `record_id, source_revision, prompt_version, input_hash` | Proveniência para rejeitar resultado antigo. Em TransactionDraft, record_id aponta para a transação recém-criada em estado Draft. |
| `idempotency_key, provider_request_id, failure_code` | Chave única, cobrança ambígua rastreável e erro sanitizado. |

## `classification_suggestions`

| Campo ou grupo | Contrato |
| --- | --- |
| `run_id, record_id, status` | SuggestionStatus e referência ao registro cuja transação ou item receberá sugestões de tags. |
| `proposed_tags` | Tabela filha classification_suggestion_tags com tag_id; novo nome sugerido fica separado e exige confirmação. |
| `proposed_new_tag_names` | Tabela filha `classification_suggestion_new_tags` com `proposed_name` e `normalized_name`, sem FK de tag até a aceitação. Confirmar explicitamente; então criar a tag e sua atribuição ao registro na mesma transação, verificando duplicidade. |
| `reviewed_by_user_id, reviewed_at, accepted_policy_revision, reason` | Revisão humana ou aceite automático sob política autorizada; actor de sistema não finge revisão humana. Sem cadeia de pensamento do modelo. |

## `calendar_command_proposals` — proposta para agenda

| Campo ou grupo | Contrato |
| --- | --- |
| `run_id, user_id, space_id, status, expires_at` | Uma proposta vinculada à execução e ao espaço autorizado; `SuggestionStatus` e validade curta. |
| `requested_action` | `CalendarCommandAction`: Create, Update, Cancel ou Delete. Cancelar preserva o histórico; excluir move para a lixeira. |
| `target_event_id, target_revision` | Nulos para criação; obrigatórios para edição, cancelamento e exclusão. A migration 17 adiciona a FK após criar `calendar_events`. Revisão divergente torna a proposta stale. |
| `proposed_title, starts_at, ends_at, starts_on, ends_on, all_day, timezone, place_id, notes, recurrence_spec` | Campos tipados e validados; não persistir payload arbitrário do modelo. Datas de dia inteiro permanecem separadas de instantes. |
| `reviewed_by_user_id, reviewed_at, automatic_policy_revision, applied_at, failure_code` | Autor e modo da decisão, aplicação idempotente e falha sanitizada. Criação automática só pode ocorrer após preferência explícita e autorização separada. |

Um pedido ambíguo não gera proposta aplicável: solicita esclarecimento ou registra candidatos identificáveis para a pessoa escolher. Editar, cancelar e excluir exigem confirmação mostrando o alvo e a mudança. Se a proposta expirar ou a agenda mudar, não aplicar automaticamente.

## `financial_transactions` — execução de `TransactionDraft`

Não criar uma segunda tabela de proposta financeira: o resultado aprovado pelo validador nasce como `financial_transactions.state=draft`, com seus itens conhecidos, e `ai_runs.record_id` guarda a proveniência. A chave idempotente do run impede criar outro rascunho em repetição. A pessoa edita esse mesmo rascunho e chama ConfirmTransaction pelo fluxo financeiro normal; descarte usa a regra de rascunho existente.

TransactionDraft nunca cria `fiscal_documents`. Não exigir itens quando a pessoa só informou descrição/total; se houver linhas, seus totais devem reconciliar. Pagamentos não são criados nem efetivados pela tarefa. O campo `party_id` ou referências de produto só recebem IDs resolvidos no mesmo espaço, com revisão de correspondência quando houver ambiguidade. A confirmação humana e o histórico permanecem na auditoria financeira.

## `ai_usage_entries`

| Campo ou grupo | Contrato |
| --- | --- |
| `run_id, attempt_number, budget_period_id, reserved_minor, actual_cost_minor` | UNIQUE(run_id, attempt_number); custo desconhecido é nulo com estado explícito. |
| `input_tokens, output_tokens, price_version, currency` | Contadores opcionais e referência de preço; não assumir zero quando ausentes. |

## `ai_budget_periods`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id, connection_id, starts_at, ends_at` | Janela de orçamento, UNIQUE(user_id, connection_id, starts_at); término posterior ao início. |
| `limit_minor, reserved_minor, spent_minor, currency` | Valores exatos; reserva/acerto sob bloqueio. Uso desconhecido mantém reserva até reconciliação ou decisão explícita. |

## Extensão opcional futura: Assistente pessoal do James

Estas tabelas são propostas para uma fase posterior à IA inicial. Não fazem parte do primeiro schema migrável; criar somente após aprovar fontes, consentimento, retenção, provedor e critérios de qualidade. A extensão do PostgreSQL `vector`/pgvector fica opcional. Começar comparando busca SQL/textual e busca vetorial exata; índice aproximado HNSW só após medir volume, latência e cobertura. Dimensão `N` e modelo não são fixados até escolher provedor/modelo.

### `ai_conversations` e `ai_messages`

Conversas pertencem ao mesmo `user_id`/`space_id`. Mensagens guardam papel, conteúdo visível, instante e referência opcional ao `ai_run_id`; estado de retenção e arquivamento ficam explícitos. Não persistir chain-of-thought, credenciais, prompts internos completos ou payloads brutos de ferramentas. Retenção de conversas tem política própria e não segue a retenção vitalícia do Activity Log.

### `ai_memory_sources`

Uma linha por Record/revisão e perfil de indexação: `space_id`, `record_id`, `source_revision`, `content_hash`, `module_code`, estado (`excluded`, `pending`, `indexed`, `failed`, `stale`), `indexing_profile`, `embedding_model`, `chunker_version` e instantes. UNIQUE na fonte/revisão/perfil. Inclusão nasce de preferência explícita por fontes/domínios; remoção, exclusão, mudança de acesso ou nova revisão agenda invalidação/reindexação idempotente.

### `ai_memory_chunks`

Uma linha por trecho com `space_id`, `memory_source_id`, `chunk_index`, snapshot mínimo de texto recuperável, `embedding vector(N)`, `embedding_model`, dimensão, versão do chunker, hash e contagem de tokens. UNIQUE por fonte/revisão/perfil/trecho. Texto, vetores e metadados derivados são dados privados e sensíveis; manter referência à fonte para abrir, corrigir, invalidar e apagar. Não transformar vetor em fonte de verdade nem carregar conteúdo de um Record não autorizado.

O mesmo conjunto/index usa uma dimensão/modelo compatíveis; troca de modelo cria uma versão de perfil e reindexação, sem misturar vetores incompatíveis. Filtro de `space_id`, fontes selecionadas e Policies vem do servidor e é aplicado antes da montagem do contexto. RLS/consulta escopada garante confidencialidade; índice ANN não é controle de autorização. Com filtros em índices aproximados, medir cobertura contra busca exata e ajustar a consulta/particionamento se necessário.

### `assistant_action_proposals` (se a camada genérica for aprovada)

Campos candidatos: `run_id`, `tool_code` allowlisted, `target_record_id`, `target_revision`, `schema_version`, parâmetros tipados validados, status, expiração, idempotency key, confirmação (`reviewed_by_user_id`, `reviewed_at`) e execução (`applied_at`). O servidor chama Action do módulo dono do dado e revalida Policy, estado e revisão. Essa tabela não deve duplicar `calendar_command_proposals` nem `financial_transactions` Draft; decidir se será apenas um envelope de orquestração ligado aos contratos já existentes.

## Integridade, índices e comportamento

- Adicionar ai_run_id nullable a tag_assignments após criar ai_runs. Comparar revisão e validar IDs na Action de aceite.
- Adicionar ai_run_id nullable a fiscal_document_extractions e uma FK composta após criar ai_runs. O job fiscal só anexa sua tentativa à execução do mesmo espaço e documento.
- A migration 17 cria a FK de `calendar_command_proposals.target_event_id`; a Action também valida que o evento e a execução pertencem ao mesmo usuário/espaço.
- User secret não entra em payload serializado de Job; o worker carrega conexão autorizada por ID. Não alterar config global mutável para trocar chaves entre usuários.
- Orçamento é reservado sob bloqueio antes da chamada; manter histórico de tentativas, sem cobrança dupla invisível.
- Uma execução libera/acerta sua reserva uma única vez. Contadores do período são reconciliáveis pelas entradas de uso; verificar limites da instalação e do usuário quando ambos existirem.
- Usar a tarefa FiscalExtraction somente como fallback permitido após o leitor estruturado/UF aplicável. Salvar processador, versão, custo e estado no histórico de extração fiscal; não guardar prompt/imagem integral em logs de IA.
- CalendarCommand só recebe a mensagem explícita e o contexto mínimo de eventos candidatos quando necessário; nunca envia o calendário inteiro por padrão nem chama integração externa por conta própria.
- TransactionDraft só recebe a descrição explícita e contexto mínimo de contraparte/produto autorizado. IDs e valores passam por validação no servidor; pedidos repetidos usam idempotência. Nunca escolher ou efetivar conta, cartão, pagador, categoria ou pagamento por inferência.
- Chave geral é opcional, habilitada por administrador e escolhida pelo usuário; sem fallback silencioso. Não incluir segredos em auditoria/exportação.
- Para o assistente futuro, recuperação SQL/textual/vetorial devolve IDs de fonte e trechos autorizados; o modelo não recebe conexão SQL, permissões globais ou capacidade de ampliar `space_id`.
- Material derivado obedece à revisão/remoção da fonte. Jobs de indexação comparam `source_revision` antes/depois da geração para evitar persistir embeddings atrasados; repetição de job não cria chunks duplicados.
- Uma resposta conversacional guarda referências de origem e versão de busca/modelo; falta de evidência produz abstenção. Conteúdo recuperado é dado não confiável, nunca instrução para trocar políticas ou ferramentas.
- Separar limites/custo de geração, embeddings e reindexação. Indexar apenas registros incluídos pelo usuário; localização, diário íntimo, saúde e finanças exigem escolha própria e não são indexados por padrão.

## Conversão do James 1.x

Não há IA no 1.x para migrar. Tags e escolhas humanas viram base de regras locais; não chamar modelo durante importação.

## Componentes previstos

SuggestClassification; ReviewSuggestion; DraftCalendarCommand; ReviewCalendarCommand; ApplyCalendarCommand; DraftTransactionFromText; ReviewAiTransactionDraft; ResolveAiConnection; ReserveAiBudget; AiProviderAdapter; reconciliar uso e falhas. Extensão futura: IndexAssistantMemory; RetrieveAssistantEvidence; AnswerWithSources; ReviewAssistantAction.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
