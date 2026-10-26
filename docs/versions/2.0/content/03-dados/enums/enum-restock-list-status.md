---
id: enum-restock-list-status
title: Enum — RestockListStatus
description: Estados de uma lista pessoal de reposição.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, estoque
related: planejamento-2-0, enums, migrations, estoque-despensa, migration-23-stock-operations
source_refs:
updated: 2026-10-26
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| Draft | draft | Preparação privada da lista. |
| Open | open | Lista em uso, editável e com recebimentos parciais possíveis. |
| Completed | completed | Pessoa concluiu a lista; linhas não recebidas ficam visíveis no histórico. |
| Archived | archived | Lista fora do uso diário, sem apagar compras ou movimentos relacionados. |

## Regras

Completed não afirma que cada linha foi comprada/armazenada e não altera saldo. Reabrir é uma transição auditada; recebimentos parciais permanecem ligados às linhas.

## Persistência

Aplicar em stock_restock_lists.status conforme Migration 23.
