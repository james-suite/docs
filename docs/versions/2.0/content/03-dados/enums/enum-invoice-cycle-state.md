---
id: enum-invoice-cycle-state
title: Enum — InvoiceCycleState
description: Valores e invariantes propostos de InvoiceCycleState.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, cartoes
source_refs:
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Open` | `open` | Ciclo aberto |
| `Closed` | `closed` | Ciclo fechado |
| `Cancelled` | `cancelled` | Ciclo cancelado |

## Regras e transições

open → closed; reabertura controlada gera auditoria. Cancelamento exige ausência de efeitos financeiros ou reversão compatível. Quitação e atraso são calculados em dimensões separadas.

## Persistência

Aplicar em `credit_card_invoices` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:cartoes) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
