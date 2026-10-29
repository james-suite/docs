---
id: migration-17-calendar
title: Migration 17 — Agenda e séries
description: Contrato lógico de tabelas, FKs, índices e conversão: agenda e séries.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, agenda-tarefas, componentes-tecnicos
source_refs: https://brasilapi.com.br/docs
updated: 2026-10-28
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 13, 16. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `calendar_series`

| Campo ou grupo | Contrato |
| --- | --- |
| `title, timezone, recurrence_spec, revision` | Regra versionada e subconjunto de repetição validado. |
| `starts_at ou starts_on` | Instante ou data de dia inteiro, com tipo explícito. |

## `calendar_events`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, series_id, original_occurrence_key` | Identidade e exceção de série opcional. |
| `title, state, starts_at, ends_at, starts_on, ends_on, all_day` | CalendarEventState; datas de dia inteiro separadas de instantes, com coerência por CHECK. |
| `place_id, notes, cancelled_at` | Lugar e observações; FKs adicionadas após pacote 13. |

## Integridade, índices e comportamento

- UNIQUE(space_id, series_id, original_occurrence_key) quando série presente.
- Data de dia inteiro não sofre conversão UTC que altere o dia. Fim deve ser posterior ou igual conforme formato definido.
- Editar esta ocorrência e próximas ocorrências são operações diferentes; preservar histórico e lembretes.
- Feriados nacionais/estaduais consultados pela BrasilAPI são uma camada de leitura/cache, não `calendar_events` pessoais; a integração é contextual, não altera vencimentos e não promete feriados municipais.

## Conversão do James 1.x

Módulo novo. Datas financeiras aparecem como projeção/ligação; não copiar cada vencimento para um evento mutável independente.

## Componentes previstos

SaveCalendarEvent; SplitCalendarSeries; CancelOccurrence; ReminderPlanner.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
