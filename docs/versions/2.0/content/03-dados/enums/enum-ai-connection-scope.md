---
id: enum-ai-connection-scope
title: Enum — AiConnectionScope
description: Valores e invariantes propostos de AiConnectionScope.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, ia-e-memoria
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Personal` | `personal` | Credencial do usuário |
| `Instance` | `instance` | Conexão oferecida pela instalação |

## Regras e transições

personal exige owner_user_id; instance exige autorização e limites por usuário. Não há chave compartilhada entre usuários como fallback implícito.
