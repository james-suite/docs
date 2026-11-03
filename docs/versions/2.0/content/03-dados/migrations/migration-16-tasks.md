---
id: migration-16-tasks
title: Migration 16 — Tarefas e checklists
description: Contrato lógico de tabelas, FKs, índices e conversão: tarefas e checklists.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, agenda-tarefas, componentes-tecnicos
source_refs:
updated: 2026-11-01
---

## `tasks`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, title, status, due_at, completed_at` | TaskStatus e marcos; prazo opcional. |
| `related_record_id, maintenance_plan_id` | Alvo autorizado e plano de manutenção opcional. |
| `priority, notes` | Prioridade validada e contexto. |

## `checklist_templates / checklist_template_items`

| Campo ou grupo | Contrato |
| --- | --- |
| `name, revision / template_id, position, label, requires_reason` | Modelo versionado; executar clona os itens. |

## `checklists / checklist_items`

| Campo ou grupo | Contrato |
| --- | --- |
| `task_id, template_revision / checklist_id, position, label, status` | Instância com labels congelados e ChecklistItemStatus. |
| `completed_at, completed_by_user_id, reason` | Histórico de conferência; reabertura auditada. |

## Integridade, índices e comportamento

- Concluir uma tarefa de manutenção não registra automaticamente um serviço; a Action deve exigir o evento quando se tratar de execução real.
- Template alterado não muda checklist já executado. Item não aplicável pode exigir motivo.
- Índices por espaço/status/prazo; relógio usa timezone da pessoa e persiste instantes UTC.

## Conversão do James 1.x

Sem tabelas equivalentes no 1.x. Criar tarefas de migração somente por decisão, sem notificações externas automáticas.

## Componentes previstos

CreateChecklistFromTemplate; CompleteTask; CompleteMaintenanceTask; ReopenChecklistItem.
