---
id: enum-capture-source
title: Enum — CaptureSource
description: Valores e invariantes propostos de CaptureSource.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, lugares-checkins
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Manual` | `manual` | Informado pela pessoa no site |
| `BrowserLocation` | `browser_location` | Posição pontual obtida pelo navegador durante salvamento/check-in iniciado pela pessoa e após permissão explícita |
| `Import` | `import` | Importação pelo site |

## Regras e transições

Guardar método de captura separado da certeza do fato. `BrowserLocation` só é usado quando a pessoa inicia uma ação no site, habilitou captura no salvamento (ou acionou o check-in) e o navegador fornece uma posição. O case se aplica a `visits` e `activity_location_contexts`; não significa que um log prove presença no lugar associado ao registro.
