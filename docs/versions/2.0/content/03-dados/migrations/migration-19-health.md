---
id: migration-19-health
title: Migration 19 — Saúde e treinos
description: Contrato lógico de tabelas, FKs, índices e conversão: saúde e treinos.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, saude-treinos, componentes-tecnicos
source_refs:
updated: 2026-10-25
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 16. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `exercises / workout_routines`

| Campo ou grupo | Contrato |
| --- | --- |
| `name, instructions / name, revision` | Catálogo pessoal de exercícios e rotina. |

## `routine_exercises`

| Campo ou grupo | Contrato |
| --- | --- |
| `routine_id, exercise_id, position, prescribed_sets` | Prescrição com schema validado; unidades explícitas. |

## `workout_sessions`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, routine_id, routine_revision, performed_at` | Sessão real e origem. |
| `routine_snapshot, notes` | Snapshot da estrutura praticada, independente de edições futuras. |

## `workout_sets`

| Campo ou grupo | Contrato |
| --- | --- |
| `session_id, exercise_id, position, repetitions, load, load_unit, duration_seconds` | Resultados tipados, não JSON sem formato. |

## `measurement_types / health_observations`

| Campo ou grupo | Contrato |
| --- | --- |
| `code, label, allowed_unit / record_id, type_id, numeric_value, unit, observed_at` | Catálogo de medição e valor com unidade compatível. |
| `source, notes` | Procedência e comentário privados. |

## Integridade, índices e comportamento

- Nenhum dado de saúde é enviado à IA sem consentimento específico.
- Rotina editada não muda sessões antigas; exercícios e unidades preservam referências históricas.
- Não usar medições como diagnóstico automático nem completar valores ausentes por inferência.

## Conversão do James 1.x

Módulo novo. Integração Apple Health fica para fase móvel, sem campos nulos espalhados em users/contacts.

## Componentes previstos

RecordWorkout; ReviseRoutine; RecordHealthObservation; formatadores de quantidade/unidade.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
