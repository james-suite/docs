---
id: enum-tag-creation-origin
title: Enum — TagCreationOrigin
description: Origem de criação de uma tag no catálogo pessoal.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, tags, migration-12-ai
source_refs:
updated: 2026-10-31
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no James. Cases usam TitleCase e valores persistidos estáveis.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Manual` | `manual` | Tag criada pela pessoa no catálogo ou durante o preenchimento de um registro. |
| `Ai` | `ai` | Tag específica criada como parte da aplicação autorizada de uma classificação; registra `created_by_ai_run_id`. |
| `Import` | `import` | Tag criada por conversão/importação de dados, com referência à execução de importação quando disponível. |

## Regras

A origem da criação não substitui a origem de cada vínculo: uma tag criada por IA pode receber atribuições posteriores manuais, por regra ou por importação. Aplicar a classificação cria a tag ausente e sua primeira atribuição na mesma operação lógica, sem confirmação individual. Reutilizar nome canônico ou alias equivalente em vez de criar duplicata. Arquivar ou desfazer uma tag criada por IA somente quando não houver outros vínculos.

Persistir a enum em `tags.creation_origin` conforme [Migration 04 — Tags](doc:migration-04-tags); a FK da execução de IA entra no pacote 12.
