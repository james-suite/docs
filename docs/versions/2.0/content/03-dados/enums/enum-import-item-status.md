---
id: enum-import-item-status
title: Enum — ImportItemStatus
description: Valores e invariantes propostos de ImportItemStatus.
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
| `Pending` | `pending` | Não aplicado |
| `Imported` | `imported` | Convertido |
| `Skipped` | `skipped` | Ignorado com motivo |
| `Failed` | `failed` | Falha |

## Regras e transições

Um item só pode ser ignorado por regra explícita apresentada na prévia. completed do pacote exige ausência de falha bloqueante; itens importados não são reaplicados em retomada.
