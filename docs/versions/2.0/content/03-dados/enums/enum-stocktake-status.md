---
id: enum-stocktake-status
title: Enum — StocktakeStatus
description: Estados de uma sessão de contagem física do estoque.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, estoque
related: planejamento-2-0, enums, migrations, estoque-despensa, migration-22-stock
source_refs:
updated: 2026-10-26
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| Draft | draft | Escopo criado, contagem ainda não iniciada. |
| Counting | counting | Coleta das quantidades observadas. |
| Review | review | Divergências e movimentos concorrentes aguardam conferência. |
| Completed | completed | Ajustes aprovados e movimento de reconciliação criado. |
| Cancelled | cancelled | Sessão encerrada sem ajustar saldo. |

## Regras

Completed e Cancelled são terminais. Completar a contagem não apaga movimentos ocorridos durante a sessão; reconcilia o saldo esperado antes de criar o ajuste.

## Persistência

Aplicar em stocktake_sessions.status. Mudança de estado, autoria e conclusão ficam na auditoria.
