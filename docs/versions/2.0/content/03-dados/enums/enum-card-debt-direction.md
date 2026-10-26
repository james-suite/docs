---
id: enum-card-debt-direction
title: Enum — CardDebtDirection
description: Valores e invariantes propostos de CardDebtDirection.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, cartoes
source_refs:
updated: 2026-10-25
---

## Contrato proposto

Enum string planejado, ainda sem classe PHP no 2.0. Cases em TitleCase e valores persistidos estáveis.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Increase` | `increase` | Aumenta dívida do cartão |
| `Decrease` | `decrease` | Crédito/devolução que reduz dívida |

## Regras

amount_minor é positivo; direção define efeito no passivo e nas parcelas. Increase/Decrease não são os lados Debit/Credit do livro interno. Devolução aponta à compra original, respeita valor remanescente e pode afetar outro ciclo.

## Persistência e implementação

Usar em `card_charges` com cast Eloquent e validação/constraints. Aplicar no fluxo de [domínio](doc:cartoes); a presença de um case não habilita automaticamente uma funcionalidade.

- [ ] Confirmar regras e transições com exemplos.
- [ ] Criar enum, rótulos e cast na implementação de 2027.
- [ ] Validar valores inválidos, escopo e conversão de datas/valores aplicável.
