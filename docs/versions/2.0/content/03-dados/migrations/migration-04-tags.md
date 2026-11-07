---
id: migration-04-tags
title: Migration 04 — Tags, categorias e regras locais
description: Contrato lógico de tabelas, FKs, índices e conversão: tags, categorias e regras locais.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, tags, componentes-tecnicos
source_refs:
updated: 2026-11-07
---

## `tags`

| Campo ou grupo | Contrato |
| --- | --- |
| `id, space_id, display_name` | Catálogo pessoal; espaço novo não recebe tags semeadas, exemplos compartilhados ou defaults. Nomes normalizados são reservados na tabela tag_names para garantir unicidade também entre aliases. |
| `description, color, icon, archived_at` | Apresentação e arquivamento; valores de estilo validados. |
| `creation_origin, created_by_user_id, created_by_ai_run_id` | TagCreationOrigin e autoria; o ID de execução é nullable e sua FK entra após criar `ai_runs` no pacote 12. |

## `tag_names`

| Campo ou grupo | Contrato |
| --- | --- |
| `tag_id, name, normalized_name, is_canonical` | FK composta; UNIQUE(space_id, normalized_name) reserva nomes canônicos e aliases no mesmo namespace. |
| `normalization_version` | Algoritmo de normalização; mudanças futuras exigem análise de colisões. |

## `tag_assignments`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, tag_id` | FKs compostas; UNIQUE(space_id, record_id, tag_id). |
| `origin, applied_by_user_id, automatic_policy_revision` | TagAssignmentOrigin, autor se houver ação manual e política que autorizou aplicação automática; não inventar revisor humano. |
| `source_revision, ai_run_id` | Revisão usada na classificação e, após o pacote 12, execução que originou a atribuição. |

## `financial_categories`

| Campo ou grupo | Contrato |
| --- | --- |
| `parent_id, name, normalized_name` | Hierarquia sem ciclos; unicidade entre irmãs incluindo raiz por índice parcial apropriado. |
| `transaction_kind, archived_at` | Income/Expense e arquivamento. Operações usuais validam a direção da categoria. Refund usa a natureza da operação original para classificar sua redução. |

## `classification_rules`

| Campo ou grupo | Contrato |
| --- | --- |
| `priority, enabled, party_id` | Regra ordenada; fornecedor opcional. |
| `conditions, result_category_id` | JSONB com schema e operadores permitidos; categoria FK. |
| `result_tags` | Tabela filha classification_rule_tags com rule_id/tag_id únicos, sem IDs soltos em texto. |

## Integridade, índices e comportamento

- A FK opcional `created_by_ai_run_id` em `tags`, `ai_run_id` em `tag_assignments` e o registro dos resultados de classificação entram no pacote 12, depois de `ai_runs` existir.
- Normalização de nomes tem algoritmo definido e versionado; índice é garantia final contra concorrência.
- Provisionar um espaço sem criar linhas em `tags` ou `tag_names`. Seeders de desenvolvimento/teste podem usar factories isoladas; nenhum seeder de produção carrega tags padrão. Importação do 1.x preserva somente o catálogo daquele usuário.
- Índice único parcial por tag onde is_canonical é verdadeiro impede dois nomes canônicos; criação/renomeação/fusão garante exatamente um na mesma transação. Label e nome canônico não podem divergir.
- Categoria financeira ampla é separada de tags específicas. Cabeçalho da transação e itens podem ter atribuições independentes; tags herdadas são apenas uma projeção de leitura e nunca criam assignments duplicados.
- Quando a classificação autorizada por IA não encontra uma tag específica adequada, criar a tag e atribuí-la no mesmo commit lógico. Sem etapa de confirmação individual; registrar `creation_origin=ai`, `ai_run_id`, atribuição e auditoria. Se um alias/termo normalizado equivalente existir, reutilizá-lo sob restrição única e concorrência.
- Uma transação sem itens mantém tags e categoria no próprio cabeçalho; os relatórios não inventam itemização nem duplicam totais.
- Fusão de tags redireciona vínculos, guarda alias e é auditada. Não apagar tag referenciada sem plano de transferência.

## Generalização e medidas de relatório

A categoria efetiva de um item é category_id ou, se ausente, default_category_id da transação. A categoria do cabeçalho cobre o valor integral somente nas transações sem itens. Ajustes globais residuais aparecem como parcela sem classificação até sua atribuição, preservando a conciliação.

Expandir descendentes por IDs e impedir ciclos em criação/movimentação de categorias. Agregar valores na categoria pai uma vez por parcela de valor. Consultas por tags podem se sobrepor e devem deduplicar o conjunto filtrado antes de calcular um total conjunto.

Chips de Party/Vehicle/Place usam referências tipadas e as respectivas relações de domínio. A UI reutiliza a apresentação dos seletores, enquanto cada provedor de opções e consulta de relatório conserva a autorização e o papel da relação.

## Conversão do James 1.x

Preservar `financial_tags` e `financial_taggables`, incluindo separação entre tags da transação e tags dos itens. `is_primary` pode gerar uma categoria correspondente com metadado legado; funções especiais viram códigos de operação, não IDs fixos.

## Componentes previstos

SaveTagsForRecord; MergeTags; ApplyClassificationRules; ApplyAiTagClassification; CreateSpecificTagForClassification; TagPicker com IDs existentes e criação manual inline.
