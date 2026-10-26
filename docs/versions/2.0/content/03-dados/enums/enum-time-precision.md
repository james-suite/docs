---
id: enum-time-precision
title: Enum — TimePrecision
description: Valores e invariantes propostos de TimePrecision.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, modelo-de-dados
source_refs:
updated: 2026-10-26
---

## Contrato proposto

Enum string planejado, ainda sem classe PHP no 2.0. Cases em TitleCase e valores persistidos estáveis.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Day` | `day` | Só o dia local é conhecido |
| `Instant` | `instant` | Instante e fuso de origem conhecidos |

## Regras

Day exige data local e deixa instante real nulo; Instant mantém ambos coerentes. created_at registra a ingestão e não aumenta a precisão do fato importado.

## Persistência e implementação

Usar em `financial_payments, journal_entries, vehicle_events` com cast Eloquent e validação/constraints. Aplicar no fluxo de [domínio](doc:modelo-de-dados); a presença de um case não habilita automaticamente uma funcionalidade.

- [ ] Confirmar regras e transições com exemplos.
- [ ] Criar enum, rótulos e cast na implementação de 2027.
- [ ] Validar valores inválidos, escopo e conversão de datas/valores aplicável.
