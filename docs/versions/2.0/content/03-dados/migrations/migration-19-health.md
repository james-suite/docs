---
id: migration-19-health
title: Migration 19 — Saúde e treinos
description: Contrato lógico de tabelas, FKs, índices e conversão: saúde e treinos.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, saude-treinos, componentes-tecnicos, enum-health-data-origin
source_refs: https://support.apple.com/en-euro/guide/iphone/iph5ede58c3d/ios
updated: 2026-11-01
---

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
| `origin, source_name, source_device, external_sample_id, source_record_hash, health_import_batch_id, source_started_at, source_ended_at` | [HealthDataOrigin](doc:enum-health-data-origin), aplicativo/aparelho externo, lote de arquivo e instante/período da amostra; lote é nulo para registros manuais e valores manuais nunca são substituídos por importações. |
| `notes` | Comentário privado separado da proveniência. |

## Importação opcional do Apple Saúde por XML

Não haverá app iOS, consulta HealthKit, endpoint móvel ou sincronização. Só avaliar uma importação manual iniciada no site depois de validar um arquivo XML real exportado pelo Apple Saúde. O arquivo é processado em staging privado, com limite de tamanho, prévia e confirmação da pessoa; o arquivo original deve ser apagado após o processamento e o prazo de staging expirar.

### `health_import_batches` (condicional à validação do formato)

| Campo ou grupo | Contrato |
| --- | --- |
| `space_id, uploaded_file_id, file_sha256, parser_version` | Escopo pessoal, referência temporária ao arquivo, hash de conteúdo e versão do parser para auditoria/reprocessamento determinístico. |
| `status, started_at, previewed_at, confirmed_at, completed_at, expires_at` | [ImportStatus](doc:enum-import-status) da importação manual; status de erro não deve guardar conteúdo de saúde nem segredos. |
| `records_seen, inserted_count, skipped_count, rejected_count, sanitized_error` | Resumo do lote para explicar a prévia e o resultado sem guardar o XML completo. |

Se o mesmo hash já foi importado, apresentar o lote anterior e não duplicar registros. Se arquivos diferentes trouxerem a mesma amostra, usar identificador externo quando presente; na falta dele, comparar tipo, unidade, valor, instante e origem, marcando possíveis duplicatas para revisão. Uma nova exportação não apaga registros antigos por ausência de uma amostra.

### `workout_sessions` e séries opcionais

- Acrescentar à sessão `HealthDataOrigin`, identificador externo quando presente, nome do app/aparelho, tipo de atividade, início/fim, duração, totais disponíveis (distância e energia), `health_import_batch_id` e fuso de origem quando fornecido.
- Amostras de frequência cardíaca e outras séries só entram após confirmar que o XML as inclui e que a resolução é útil. Campos candidatos: `space_id, workout_session_id, health_import_batch_id, metric_code, started_at, ended_at, numeric_value, unit, external_sample_id, source_name`. Índice por sessão/tipo/instante e idempotência por amostra.
- Rota GPS de treino não é garantida pelo contrato público do arquivo. Só planejar armazenamento de geometria/arquivo privado depois de validá-la em amostra real; jamais converter pontos de treino em `Visit` ou `ActivityLocationContext`.
- Não replicar amostras de séries como medições corporais nem armazenar resolução maior do que as telas realmente usam.

## Integridade, índices e comportamento

- Nenhum dado de saúde é enviado à IA sem consentimento específico.
- Rotina editada não muda sessões antigas; exercícios e unidades preservam referências históricas.
- Não usar medições como diagnóstico automático nem completar valores ausentes por inferência.
- Dados importados preservam origem, unidade e identificador/idempotência; sessões manuais e importadas convivem sem sobrescrever umas às outras. FKs do lote incluem `space_id`.
- A consulta de uma rota de exercício não cria check-in, contexto de auditoria ou localização em registro não relacionado à sessão.
- A pessoa pode exportar e excluir dados importados; apagar um lote permite remover apenas os registros cuja origem seja aquele lote, sem afetar registros manuais.

## Conversão do James 1.x

Módulo novo. Não há conversão de dados do James 1.x. A importação opcional por arquivo não espalha campos nulos por `users`/`contacts` e não é requisito da primeira importação migrável.

## Componentes previstos

RecordWorkout; ReviseRoutine; RecordHealthObservation; HealthExportUpload; HealthExportReview; WorkoutMetricChart; formatadores de quantidade/unidade. WorkoutRouteMap só entra se um arquivo validado trouxer trilha utilizável.
