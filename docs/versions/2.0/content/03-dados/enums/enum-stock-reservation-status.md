---
id: enum-stock-reservation-status
title: Enum — StockReservationStatus
description: Ciclo de vida de uma reserva de quantidade disponível.
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
| Active | active | Quantidade bloqueada para um propósito, ainda em estoque físico. |
| Fulfilled | fulfilled | Quantidade consumida ou alocada por movimento confirmado. |
| Released | released | Reserva devolvida à disponibilidade sem saída física. |
| Expired | expired | Prazo passou; disponibilidade foi liberada por transição auditada. |
| Cancelled | cancelled | Reserva cancelada pela pessoa antes de cumprir. |

## Regras

Fulfilled, Released, Expired e Cancelled são estados terminais desta reserva. Reserva nunca é movimento de estoque; cumprir consumo cria movimento e vínculo à linha reservada. Se apenas parte for cumprida, preservar quantidade e estado restante ou dividir em uma nova reserva.

## Persistência

Aplicar em stock_reservations.status conforme Migration 23.
