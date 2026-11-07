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
updated: 2026-11-07
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

uploaded → validating → ready → running → completed. Cancelar permitido antes da publicação. Validação pode avançar por lotes em staging. No importador financeiro inicial, falha na publicação reverte o domínio inteiro e permite retomar o staging. Completed exige reconciliação da publicação e arquivos disponíveis.

Validating abrange a preparação do grafo e suas revisões. Ready significa que a versão preparada atual passou no validador e pode ser confirmada. Editar a preparação retorna à validação e invalida o aceite anterior. Running publica o plano fixado pela confirmação.
