---
id: enum-health-data-origin
title: Enum — HealthDataOrigin
description: Procedência da medição ou sessão de saúde no James.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, migration-19-health, saude-treinos
source_refs:
updated: 2026-10-31
---

## Contrato proposto

Enum string da forma pela qual o dado entrou no James. O provedor/aplicativo que originalmente produziu a amostra fica em campos de proveniência separados; várias fontes externas podem alimentar o Apple Health.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Manual` | `manual` | Registro criado diretamente no James. |
| `HealthKit` | `healthkit` | Amostra enviada pelo cliente iOS depois de consultar o HealthKit. |
| `HealthExport` | `health_export` | Amostra importada de um arquivo exportado no app Saúde. |

## Regras

Manter registros manuais separados de valores importados. O valor de origem identifica o caminho de entrada, não a precisão clínica nem se a pessoa autorizou todos os tipos. O app/aparelho produtor e o identificador externo ficam em campos separados quando disponíveis.

## Persistência

Aplicar a `health_observations` e `workout_sessions` na extensão da [Migration 19](doc:migration-19-health). Um conector direto adicional pode exigir outro case; não criar enum por fornecedor enquanto a origem funcional continuar sendo HealthKit.
