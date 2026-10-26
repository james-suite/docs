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
updated: 2026-10-25
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 15. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

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

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
