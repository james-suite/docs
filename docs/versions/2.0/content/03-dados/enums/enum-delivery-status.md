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
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Queued` | `queued` | Na fila |
| `Sent` | `sent` | Aceito pelo provedor |
| `Failed` | `failed` | Falhou |
| `Skipped` | `skipped` | Envio dispensado por regra |

## Regras e transições

Tentativa queued → sent/failed/skipped. Reenvio cria nova tentativa; sent não significa lido. Falha permanente sanitizada não é reenviada infinitamente.

## Persistência

Aplicar em `notification_delivery_attempts` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:notificacoes) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
