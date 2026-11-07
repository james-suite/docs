---
id: enum-journal-state
title: Enum — JournalState
description: Valores e invariantes propostos de JournalState.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, diario
source_refs:
updated: 2026-11-07
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Draft` | `draft` | Rascunho |
| `Published` | `published` | Registrado no diário |

## Regras e transições

Arquivamento usa archived_at e preserva o estado Draft ou Published, o conteúdo e as revisões. Exclusão usa ciclo próprio; published não significa público para outras pessoas.
