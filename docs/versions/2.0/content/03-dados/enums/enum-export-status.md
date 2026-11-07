---
id: enum-export-status
title: Enum — ExportStatus
description: Estado da geração e disponibilidade de um pacote exportado.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: enums, migration-20-imports, portabilidade-backup
source_refs:
updated: 2026-11-07
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Queued` | `queued` | Solicitação aceita para geração |
| `Running` | `running` | Snapshot e arquivos em processamento |
| `Completed` | `completed` | Pacote íntegro publicado para download |
| `Failed` | `failed` | Geração interrompida com diagnóstico sanitizado |
| `Expired` | `expired` | Artefato removido após prazo ou descarte autorizado |

Queued passa a Running, e Running termina em Completed ou Failed. Completed passa a Expired após a limpeza do artefato. Retomar uma falha usa nova tentativa identificada e valida o snapshot original ou solicita outro snapshot. Publicar Completed exige todos os grupos declarados e hashes de arquivos válidos.
