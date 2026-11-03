---
id: enum-split-mode
title: Enum — SplitMode
description: Valores e invariantes propostos de SplitMode.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, acertos
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Equal` | `equal` | Igual entre consumidores selecionados |
| `Exact` | `exact` | Valores exatos |
| `Weighted` | `weighted` | Por pesos definidos |

## Regras e transições

equal não inclui o organizador automaticamente. exact aceita parcela própria zero. weighted é etapa posterior; não habilitar na UI antes do algoritmo e critérios de arredondamento.
