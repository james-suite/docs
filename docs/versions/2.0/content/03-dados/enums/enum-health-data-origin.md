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
updated: 2026-11-01
---

## Contrato proposto

Enum string da forma pela qual o dado entrou no James. O aplicativo/aparelho que originalmente produziu a amostra fica em campos de proveniência separados quando o arquivo fornece essa informação.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Manual` | `manual` | Registro criado diretamente no James. |
| `HealthExport` | `health_export` | Amostra importada de um arquivo exportado no app Saúde. |

## Regras

Manter registros manuais separados de valores importados. O valor de origem identifica o caminho de entrada, não a precisão clínica. O app/aparelho produtor e o identificador externo ficam em campos separados quando o arquivo os fornecer.

Sem uma fonte de importação validada, manter apenas registros manuais; não criar cases para integrações que dependam de app iOS.
