---
id: enum-fiscal-extraction-status
title: Enum — FiscalExtractionStatus
description: Estado de processamento e revisão de dados lidos de um documento fiscal.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas, migration-06-transactions
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Pending` | `pending` | Captura aguardando processamento. |
| `Processing` | `processing` | Job de leitura em execução. |
| `NeedsReview` | `needs_review` | Há candidatos, mas precisam de conferência humana. |
| `Accepted` | `accepted` | Usuário aceitou os campos extraídos; a transação ainda pode estar em rascunho. |
| `Failed` | `failed` | Leitura falhou; o usuário ainda pode preencher manualmente. |

## Regras e transições

Processamento repetido precisa ser idempotente e uma tentativa nova não pode sobrescrever campos financeiros já confirmados. A falha de OCR não bloqueia cadastro manual. `Accepted` significa somente que a extração foi revisada; não efetiva pagamento nem confirma a transação.
