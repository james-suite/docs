---
id: enum-health-sync-status
title: Enum — HealthSyncStatus
description: Estados propostos de uma importação de dados de saúde.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, migration-19-health, saude-treinos
source_refs: https://developer.apple.com/documentation/healthkit/authorizing-access-to-health-data
updated: 2026-10-31
---

## Contrato proposto

Enum string para `health_sync_runs`; não representa a permissão concedida pelo usuário no Apple Health. O HealthKit não informa de forma confiável se o acesso de leitura a cada tipo foi recusado, portanto não criar cases `Authorized`/`Denied`.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Running` | `running` | Lote iniciado e ainda não finalizado. |
| `Completed` | `completed` | Lote processado sem erros locais; não afirma que toda a janela HealthKit estava visível. |
| `PartiallyCompleted` | `partially_completed` | Registros recebidos e processados, com itens/arquivos rejeitados identificados. |
| `Failed` | `failed` | O lote não foi processado; erro sanitizado e possibilidade de tentar novamente. |
| `Cancelled` | `cancelled` | A pessoa cancelou antes de concluir o envio/processamento. |

## Regras e transições

`Running` termina em `Completed`, `PartiallyCompleted`, `Failed` ou `Cancelled`. Reenvio com a mesma chave não duplica dados. Ausência de amostras não prova permissão negada nem inexistência do dado e não é, sozinha, uma execução parcial.

## Persistência

Aplicar em `health_sync_runs` na extensão opcional da [Migration 19](doc:migration-19-health). Persistir valores técnicos estáveis, nunca mensagens exibidas. Se futuramente outros conectores tiverem exatamente o mesmo ciclo, avaliar um tipo compartilhado; não generalizar antes de existir segundo provedor.
