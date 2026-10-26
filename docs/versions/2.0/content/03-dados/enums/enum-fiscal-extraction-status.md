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
updated: 2026-10-26
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Pending` | `pending` | Captura aguardando processamento. |
| `Processing` | `processing` | Job de leitura em execução. |
| `NeedsReview` | `needs_review` | Há candidatos, mas precisam de conferência humana. |
| `Accepted` | `accepted` | Usuário aceitou os campos extraídos; a transação ainda pode estar em rascunho. |
| `Failed` | `failed` | Leitura falhou; o usuário ainda pode preencher manualmente. |

## Regras e transições

Processamento repetido precisa ser idempotente e uma tentativa nova não pode sobrescrever campos financeiros já confirmados. A falha de OCR não bloqueia cadastro manual. `Accepted` significa somente que a extração foi revisada; não efetiva pagamento nem confirma a transação.

## Persistência

Aplicar em `fiscal_documents` e `fiscal_document_extractions` conforme [Migration 06](doc:migration-06-transactions).

## Checklist

- [ ] Revisar estados no fluxo de [finanças](doc:financas).
- [ ] Criar enum e rótulos no Model correspondente.
- [ ] Testar repetição de Jobs, erro de leitura, revisão e aceitação sem efeitos financeiros implícitos.
