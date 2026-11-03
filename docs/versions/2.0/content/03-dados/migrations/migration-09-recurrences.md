---
id: migration-09-recurrences
title: Migration 09 — Recorrências e ocorrências
description: Contrato lógico de tabelas, FKs, índices e conversão: recorrências e ocorrências.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, financas, componentes-tecnicos
source_refs:
updated: 2026-11-01
---

## `financial_recurrences`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, kind, amount_minor, currency` | Identidade, direção e valor padrão. |
| `frequency, interval, anchor_on, timezone` | Frequência, intervalo positivo, âncora e fuso original. |
| `ends_on, paused_at, archived_at` | Limites e suspensão; não apagar ocorrências passadas. |
| `default_account_id, default_card_id` | Instrumento sugerido, com regra XOR quando preenchido. |

## `recurrence_templates`

| Campo ou grupo | Contrato |
| --- | --- |
| `recurrence_id, revision, payload_version, payload` | Snapshot imutável do template de itens/categorias/tags; schema validado com IDs do mesmo espaço. |

## `recurrence_occurrences`

| Campo ou grupo | Contrato |
| --- | --- |
| `recurrence_id, scheduled_on, template_revision` | Ocorrência nominal única por regra/data. |
| `transaction_id, skipped_at, generated_at` | Resultado ou exclusão explícita; nunca gerar de novo uma ocorrência já processada. |

## Integridade, índices e comportamento

- Adicionar FK de recorrência às transações apenas agora, após criar as tabelas; DDL separada do importador.
- UNIQUE(space_id, recurrence_id, scheduled_on). Bloqueio e unicidade no banco são a garantia; cache/job único são auxiliares.
- Datas com dia 31 usam âncora original e clamp por mês, sem deriva para dia 28 nos meses seguintes.
- Editar template afeta próximas ocorrências; não sobrescrever transação manualmente revisada. Instrumentos arquivados geram pendência de configuração.

## Conversão do James 1.x

Preservar recorrências e datas já materializadas. Popular chaves de ocorrência antes de ativar scheduler para não gerar cópias de transações importadas.

## Componentes previstos

GenerateOccurrence; PauseRecurrence; SkipOccurrence; PreviewNextDates; scheduler reprocessável por intervalo.
