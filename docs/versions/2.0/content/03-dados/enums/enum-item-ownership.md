---
id: enum-item-ownership
title: Enum — ItemOwnership
description: Valores e invariantes propostos de ItemOwnership.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, acertos
source_refs:
updated: 2026-11-07
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Personal` | `personal` | Consumo próprio |
| `ThirdParty` | `third_party` | Valor de terceiro |
| `Mixed` | `mixed` | Consumo compartilhado que depende de cotas |
| `Unknown` | `unknown` | Consumo ainda sem atribuição confirmada |

## Regras e transições

A divisão permite separar despesa pessoal e adiantamento. Itens de terceiro devem explicar sua relação com cotas/obrigações; não inferir por nome de tag.

Usar também no cabeçalho para uma transação sem itens. Quem pagou permanece independente de quem consumiu. Quando houver rateio, as cotas confirmadas são fonte do valor de consumo por pessoa. Ownership descreve o contexto e é validado contra essas cotas.

Um rateio definido apenas pelo total pode conhecer o consumo pessoal agregado e deixar a distribuição por categoria/item pendente. Gráficos detalhados expõem essa cobertura. Qualquer distribuição proporcional analítica recebe indicação de estimativa e não materializa cotas ou dívidas.
