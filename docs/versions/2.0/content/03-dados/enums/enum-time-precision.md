---
id: enum-time-precision
title: Enum — TimePrecision
description: Valores e invariantes propostos de TimePrecision.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, modelo-de-dados
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Day` | `day` | Só o dia local é conhecido |
| `Instant` | `instant` | Instante e fuso de origem conhecidos |

## Regras

Day exige data local e deixa instante real nulo; Instant mantém ambos coerentes. created_at registra a ingestão e não aumenta a precisão do fato importado.
