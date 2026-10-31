---
id: migration-19-health
title: Migration 19 — Saúde e treinos
description: Contrato lógico de tabelas, FKs, índices e conversão: saúde e treinos.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, saude-treinos, componentes-tecnicos, enum-health-sync-status, enum-health-data-origin
source_refs: https://developer.apple.com/documentation/healthkit, https://developer.apple.com/documentation/healthkit/authorizing-access-to-health-data, https://developer.apple.com/documentation/healthkit/reading-route-data, https://developer.apple.com/documentation/healthkit/adding-samples-to-a-workout, https://support.apple.com/en-euro/guide/iphone/iph5ede58c3d/ios
updated: 2026-10-31
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
| `origin, source_name, source_device, external_sample_id, health_connection_id, health_sync_run_id, import_run_id, source_started_at, source_ended_at` | [HealthDataOrigin](doc:enum-health-data-origin), aplicativo/aparelho externo, execução de origem e instante/período da amostra; conexões/execuções são nulas para registros manuais e valores manuais nunca são substituídos por importações. |
| `notes` | Comentário privado separado da proveniência. |

## Extensão opcional: Apple Health/HealthKit

O James web não acessa o armazenamento local do HealthKit. A integração completa requer um app iOS com a capability e permissões HealthKit; uma importação XML manual pelo navegador é alternativa de carga/contingência, não um sincronizador equivalente. A ponte iOS propõe um envio autenticado, por ação da pessoa, limitado a dados de saúde e separado do fluxo de check-in/localização do site. Sem execução em segundo plano no primeiro corte. XML usa um fluxo de importação de arquivo privado com staging e limpeza após processamento, sem exigir uma conexão HealthKit.

Estender esta Migration 19, sem adicionar pacote de domínio independente:

### `health_connections`

| Campo ou grupo | Contrato |
| --- | --- |
| `space_id, provider_key, device_instance_id, label` | Conexão revogável da instalação iOS ao espaço pessoal; `provider_key` extensível (primeiro `apple_healthkit`), sem credenciais de API de terceiros no registro. `UNIQUE(space_id, provider_key, device_instance_id)`. |
| `created_at, last_seen_at, last_sync_at, revoked_at` | Ciclo da conexão e atividade técnica mínima. Não registrar um estado de permissão de leitura que o HealthKit não permita verificar. |
| `credential_reference` | Referência a credencial curta, revogável e limitada à ingestão; guardar conforme mecanismo de autenticação escolhido em 2027, nunca registrar bearer token em claro em logs. |

### `health_sync_runs`

| Campo ou grupo | Contrato |
| --- | --- |
| `space_id, health_connection_id, status, idempotency_key` | Execução auditável da ponte HealthKit e idempotente, com `UNIQUE(space_id, idempotency_key)` e [HealthSyncStatus](doc:enum-health-sync-status). |
| `requested_from, requested_until, started_at, finished_at` | Intervalo e execução solicitados. Não declarar que uma permissão de leitura foi concedida; HealthKit não expõe essa distinção de forma confiável para alguns tipos. |
| `received_count, inserted_count, updated_count, removed_count, sanitized_error` | Resultado operacional mínimo; não guardar payload de saúde ou rota no log da execução. |

### `health_sync_run_types`

| Campo ou grupo | Contrato |
| --- | --- |
| `space_id, health_sync_run_id, health_type_code` | Um tipo solicitado por linha, como `body_mass`, `height`, `workout`, `heart_rate` ou `workout_route`; `UNIQUE(space_id, health_sync_run_id, health_type_code)`. |
| `requested_from, requested_until, received_count, sanitized_error` | Resultado por tipo para explicar cobertura sem guardar as amostras neste registro técnico. |

### `workout_sessions` importadas e `workout_metric_samples`

- Acrescentar à sessão `HealthDataOrigin`, ID externo, nome do app/aparelho, tipo de atividade, início/fim, duração, totais disponíveis (distância e energia), `health_connection_id`, `health_sync_run_id` e `import_run_id` nulos para registros manuais. Preservar fuso de origem quando fornecido.
- Criar amostras de frequência cardíaca e outras séries somente quando configuradas e autorizadas. Campos: `space_id, workout_session_id, health_sync_run_id, metric_code, started_at, ended_at, numeric_value, unit, external_sample_id, source_name`. Índice por sessão/tipo/instante e chave de idempotência externa.
- Resumo (média/mínima/máxima quando disponíveis) e amostra individual são níveis diferentes. Não replicar cada amostra como `HealthObservation` nem armazenar resolução maior do que as telas realmente usam.

### `workout_routes`

- Relação privada com `space_id` e `workout_session_id`; armazenar uma geometria/arquivo de rota em mídia privada ou formato compacto após validar volume e necessidades de consulta.
- Guardar apenas metadados necessários à exibição e reconciliação. Não duplicar pontos em audit logs, não publicar URL de arquivo e não converter rota em `Visit` ou `ActivityLocationContext`.
- Excluir/retirar a rota independentemente da sessão quando solicitado, preservando uma sessão sem coordenadas. Incluir rota, proveniência e consentimento de captura no export pessoal.

### Limites de integridade e sincronização

- UNIQUE por espaço, origem e identificador externo estável em observações/sessões/amostras quando disponível. Todas as FKs entre esses registros devem incluir `space_id`. Quando o formato de exportação não fornecer ID estável, guardar hash de origem/arquivo e apresentar possíveis duplicatas para revisão em vez de fundir automaticamente.
- Uma amostra removida na fonte pode ser marcada como removida pelo provedor ou retirada do conjunto importado. Não apagar a observação manual correspondente.
- Não somar atividades ou amostras sobrepostas de fontes diferentes sem regra de deduplicação. Preservar app/aparelho de origem para que o usuário consiga entender divergências.
- A fonte pode limitar intervalo histórico; ausência de amostras não prova ausência de dado ou recusa da permissão. Informar a janela de importação conhecida e a última sincronização.
- IDs de âncora incrementais do HealthKit permanecem no dispositivo se isso evitar guardar no servidor metadados internos do sistema. A implementação decide após protótipo de recuperação e troca de aparelho.

## Integridade, índices e comportamento

- Nenhum dado de saúde é enviado à IA sem consentimento específico.
- Rotina editada não muda sessões antigas; exercícios e unidades preservam referências históricas.
- Não usar medições como diagnóstico automático nem completar valores ausentes por inferência.
- Dados importados preservam origem, unidade e identificador/idempotência; sessões manuais e importadas convivem sem sobrescrever umas às outras.
- A consulta de uma rota de exercício não cria check-in, contexto de auditoria ou localização em registro não relacionado à sessão.
- Arquivamento/revogação da conexão interrompe novos envios sem apagar automaticamente o histórico já importado; a tela oferece exclusão/exportação separadas.

## Conversão do James 1.x

Módulo novo. Não há conversão de dados do James 1.x. A integração HealthKit/XML não espalha campos nulos por `users`/`contacts` e não é requisito da primeira importação migrável.

## Componentes previstos

RecordWorkout; ReviseRoutine; RecordHealthObservation; HealthKitConnectionSettings; HealthSyncReview; WorkoutMetricChart; WorkoutRouteMap; formatadores de quantidade/unidade. A versão do app iOS e a autenticação do conector são decisões próprias, não APIs de check-in.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
