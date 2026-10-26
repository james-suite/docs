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
updated: 2026-10-25
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 03. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `tags`

| Campo ou grupo | Contrato |
| --- | --- |
| `id, space_id, display_name` | Catálogo pessoal; nomes normalizados são reservados na tabela tag_names para garantir unicidade também entre aliases. |
| `description, color, icon, archived_at` | Apresentação e arquivamento; valores de estilo validados. |

## `tag_names`

| Campo ou grupo | Contrato |
| --- | --- |
| `tag_id, name, normalized_name, is_canonical` | FK composta; UNIQUE(space_id, normalized_name) reserva nomes canônicos e aliases no mesmo namespace. |
| `normalization_version` | Algoritmo de normalização; mudanças futuras exigem análise de colisões. |

## `tag_assignments`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, tag_id` | FKs compostas; UNIQUE(space_id, record_id, tag_id). |
| `origin, confirmed_by_user_id` | TagAssignmentOrigin e responsável pelo aceite. |
| `source_revision` | Revisão da fonte utilizada pela automação. |

## `financial_categories`

| Campo ou grupo | Contrato |
| --- | --- |
| `parent_id, name, normalized_name` | Hierarquia sem ciclos; unicidade entre irmãs incluindo raiz por índice parcial apropriado. |
| `transaction_kind, archived_at` | Income/Expense e arquivamento; não aceitar categoria incompatível. |

## `classification_rules`

| Campo ou grupo | Contrato |
| --- | --- |
| `priority, enabled, party_id` | Regra ordenada; fornecedor opcional. |
| `conditions, result_category_id` | JSONB com schema e operadores permitidos; categoria FK. |
| `result_tags` | Tabela filha classification_rule_tags com rule_id/tag_id únicos, sem IDs soltos em texto. |

## Integridade, índices e comportamento

- As sugestões de IA e a FK opcional ai_run_id das atribuições entram no pacote 12, depois de ai_runs existir.
- Normalização de nomes tem algoritmo definido e versionado; índice é garantia final contra concorrência.
- Índice único parcial por tag onde is_canonical é verdadeiro impede dois nomes canônicos; criação/renomeação/fusão garante exatamente um na mesma transação. Label e nome canônico não podem divergir.
- Uma categoria por item; várias tags. Uso herdado é calculado, sem duplicar assignments invisíveis.
- Fusão de tags redireciona vínculos, guarda alias e é auditada. Não apagar tag referenciada sem plano de transferência.

## Conversão do James 1.x

Preservar financial_tags e financial_taggables. is_primary é traduzido para categoria proposta com metadado legado; funções especiais viram códigos de operação, não IDs fixos.

## Componentes previstos

SaveTagsForRecord; MergeTags; ApplyClassificationRules; TagPicker com vários IDs e nomes pendentes resolvidos no salvamento.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
