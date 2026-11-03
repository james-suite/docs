---
id: enum-tag-assignment-origin
title: Enum — TagAssignmentOrigin
description: Valores e invariantes propostos de TagAssignmentOrigin.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, tags
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Manual` | `manual` | Escolha humana |
| `Rule` | `rule` | Regra determinística |
| `Ai` | `ai` | Classificação aplicada pela pessoa ou por política automática explicitamente autorizada; inclui tag criada e atribuída na mesma operação |
| `Import` | `import` | Informação importada |

## Regras e transições

A origem acompanha a atribuição. Uma escolha manual não é substituída por automação. Trocar a origem exige operação explícita.
