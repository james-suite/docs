---
id: enum-import-status
title: Enum — ImportStatus
description: Valores e invariantes propostos de ImportStatus.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, migracao-2-0
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Uploaded` | `uploaded` | Arquivo recebido |
| `Validating` | `validating` | Validando |
| `Ready` | `ready` | Prévia disponível |
| `Running` | `running` | Importando |
| `Completed` | `completed` | Concluída e reconciliada |
| `Failed` | `failed` | Falhou |
| `Cancelled` | `cancelled` | Cancelada |

## Regras e transições

uploaded → validating → ready → running → completed. Cancelar permitido antes da publicação. Falha após lotes gera estado recuperável; completed exige reconciliação, não apenas término do worker.
