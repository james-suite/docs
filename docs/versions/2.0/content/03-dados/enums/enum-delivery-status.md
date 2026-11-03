---
id: enum-delivery-status
title: Enum — DeliveryStatus
description: Valores e invariantes propostos de DeliveryStatus.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, notificacoes
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Queued` | `queued` | Na fila |
| `Sent` | `sent` | Aceito pelo provedor |
| `Failed` | `failed` | Falhou |
| `Skipped` | `skipped` | Envio dispensado por regra |

## Regras e transições

Tentativa queued → sent/failed/skipped. Reenvio cria nova tentativa; sent não significa lido. Falha permanente sanitizada não é reenviada infinitamente.
