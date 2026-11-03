---
id: enum-suggestion-status
title: Enum — SuggestionStatus
description: Valores e invariantes propostos de SuggestionStatus.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, ia-e-memoria, agenda-tarefas
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Proposed` | `proposed` | Aguardando revisão |
| `Accepted` | `accepted` | Aceita |
| `Rejected` | `rejected` | Recusada |
| `Stale` | `stale` | Fonte ou catálogo mudou |
| `Expired` | `expired` | Proposta não foi confirmada dentro do prazo |

## Regras e transições

proposed → accepted/rejected/stale/expired. Aceite revalida IDs, contexto e revisão da fonte; respostas antigas não são aplicadas automaticamente. Usar em sugestões de tags e propostas de comando de agenda.
