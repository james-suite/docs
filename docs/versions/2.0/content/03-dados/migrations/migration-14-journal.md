---
id: migration-14-journal
title: Migration 14 — Diário e revisões
description: Contrato lógico de tabelas, FKs, índices e conversão: diário e revisões.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, diario, documentos-arquivos, migration-22-documents, componentes-tecnicos
source_refs:
updated: 2026-11-07
---

## `journal_entries`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, title, body_markdown, state, archived_at` | Texto pessoal e JournalState. |
| `occurred_on, occurred_at, time_precision, timezone` | TimePrecision: dia ou instante; não inventar hora para relato diário. |
| `written_at, current_revision` | Momento da escrita e controle de concorrência. |

## `journal_entry_revisions`

| Campo ou grupo | Contrato |
| --- | --- |
| `entry_id, revision_number, body_markdown, title` | Snapshot de texto privado; UNIQUE(entry_id, revision_number). |
| `edited_by_user_id, edited_at` | Autoria da revisão. |

## Integridade, índices e comportamento

- Texto íntimo completo fica nas revisões protegidas, não replicado integralmente no Activity Log.
- Links e tags usam records; arquivos anexados são Documents com media privada e vínculo à entrada. Busca e futuras memórias derivadas seguem exclusão da fonte.
- Markdown sanitizado na renderização; alterações concorrentes exigem revisão em vez de sobrescrever silenciosamente.

## Conversão do James 1.x

Módulo novo; notas de contatos continuam nos contatos, sem transformá-las automaticamente em entradas de diário.

## Componentes previstos

SaveJournalEntry; RestoreJournalRevision; LinkRecords; autorização do texto, revisões e anexos.
