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
updated: 2026-11-14
---

## `journals`

| Campo ou grupo | Contrato |
| --- | --- |
| `id, space_id` | PK convencional e FK obrigatória para spaces. UNIQUE(space_id, id) permite referências compostas. Vários diários por espaço, criados pela pessoa. |
| `name, description` | Nome obrigatório e descrição opcional. IDs distinguem diários, inclusive homônimos. |
| `archived_at, created_at, updated_at` | Arquivamento preserva entradas e revisões. |
| `ai_style_reference_enabled, ai_content_reference_enabled` | Boolean false por padrão, com finalidades independentes. A execução também revalida conexão autorizada, fonte/revisão e seleção da tarefa. |

## `journal_entries`

| Campo ou grupo | Contrato |
| --- | --- |
| `journal_id` | FK obrigatória para journals, composta com space_id. Cada entrada pertence a um diário do mesmo espaço. |
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
- Criar journals antes das entradas. Preservar as FKs de record, espaço, diário, revisão e autoria. Indexar entradas por espaço/diário/data para navegação e filtros.
- Mover uma entrada valida os dois diários e suas revisões de estado, preserva record_id, revisões, anexos e vínculos e audita a mudança. Reavaliar elegibilidade de IA após a mudança e invalidar derivados afetados.
- Exclusão de diário usa RESTRICT enquanto houver entradas. Realocação e exclusão do conteúdo passam por operação explícita e transacional, conforme as políticas de retenção das fontes e arquivos.
- Busca e recuperação de IA recebem filtro de diário e finalidade autorizada. Perfis de estilo e índices conservam fonte/revisão, permitindo invalidar derivados após edição, exclusão ou revogação.
- Exportação contém journals e a FK de cada entrada. Importação reconstrói os vínculos e preserva preferências, com nova autorização para envio de conteúdo ao provedor de IA no destino.
- Links e tags usam records; arquivos anexados são Documents com media privada e vínculo à entrada. Busca e futuras memórias derivadas seguem exclusão da fonte.
- Markdown sanitizado na renderização; alterações concorrentes exigem revisão em vez de sobrescrever silenciosamente.

## Conversão do James 1.x

Módulo novo; notas de contatos continuam nos contatos, sem transformá-las automaticamente em entradas de diário.

## Componentes previstos

CreateJournal; ArchiveJournal; MoveJournalEntry; SaveJournalEntry; RestoreJournalRevision; LinkRecords; autorização do texto, revisões e anexos.
