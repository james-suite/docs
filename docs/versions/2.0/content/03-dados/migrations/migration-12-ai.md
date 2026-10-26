---
id: migration-12-ai
title: Migration 12 — IA, credenciais e sugestões
description: Contrato lógico de tabelas, FKs, índices e conversão: ia, credenciais e sugestões.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, ia-e-memoria, componentes-tecnicos
source_refs:
updated: 2026-10-26
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 04. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

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
| `feature_permissions, budget_limit_minor, budget_currency` | Consentimentos por função e orçamento. Permitir somente classificação de tags e fallback fiscal; cada função começa desativada até autorização explícita. |

## `ai_runs`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id, space_id, connection_id, task, status` | Solicitante e contexto imutáveis. |
| `record_id, source_revision, prompt_version, input_hash` | Proveniência para rejeitar resultado antigo. |
| `idempotency_key, provider_request_id, failure_code` | Chave única, cobrança ambígua rastreável e erro sanitizado. |

## `classification_suggestions`

| Campo ou grupo | Contrato |
| --- | --- |
| `run_id, record_id, status` | SuggestionStatus e referência ao registro cuja transação ou item receberá sugestões de tags. |
| `proposed_tags` | Tabela filha classification_suggestion_tags com tag_id; novo nome sugerido fica separado e exige confirmação. |
| `proposed_new_tag_names` | Tabela filha `classification_suggestion_new_tags` com `proposed_name` e `normalized_name`, sem FK de tag até a aceitação. Confirmar explicitamente; então criar a tag e sua atribuição ao registro na mesma transação, verificando duplicidade. |
| `reviewed_by_user_id, reviewed_at, accepted_policy_revision, reason` | Revisão humana ou aceite automático sob política autorizada; actor de sistema não finge revisão humana. Sem cadeia de pensamento do modelo. |

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

## Integridade, índices e comportamento

- Adicionar ai_run_id nullable a tag_assignments após criar ai_runs. Comparar revisão e validar IDs na Action de aceite.
- Adicionar ai_run_id nullable a fiscal_document_extractions e uma FK composta após criar ai_runs. O job fiscal só anexa sua tentativa à execução do mesmo espaço e documento.
- User secret não entra em payload serializado de Job; o worker carrega conexão autorizada por ID. Não alterar config global mutável para trocar chaves entre usuários.
- Orçamento é reservado sob bloqueio antes da chamada; manter histórico de tentativas, sem cobrança dupla invisível.
- Uma execução libera/acerta sua reserva uma única vez. Contadores do período são reconciliáveis pelas entradas de uso; verificar limites da instalação e do usuário quando ambos existirem.
- Usar a tarefa FiscalExtraction somente como fallback permitido após o leitor estruturado/UF aplicável. Salvar processador, versão, custo e estado no histórico de extração fiscal; não guardar prompt/imagem integral em logs de IA.
- Chave geral é opcional, habilitada por administrador e escolhida pelo usuário; sem fallback silencioso. Não incluir segredos em auditoria/exportação.

## Conversão do James 1.x

Não há IA no 1.x para migrar. Tags e escolhas humanas viram base de regras locais; não chamar modelo durante importação.

## Componentes previstos

SuggestClassification; ReviewSuggestion; ResolveAiConnection; ReserveAiBudget; AiProviderAdapter; reconciliar uso e falhas.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
