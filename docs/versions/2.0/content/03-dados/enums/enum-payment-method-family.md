---
id: enum-payment-method-family
title: Enum — PaymentMethodFamily
description: Valores e invariantes propostos de PaymentMethodFamily.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas
source_refs:
updated: 2026-10-25
---

## Contrato proposto

Enum string planejado, ainda sem classe PHP no 2.0. Cases em TitleCase e valores persistidos estáveis.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Cash` | `cash` | Dinheiro/carteira |
| `Bank` | `bank` | Fluxo bancário, como Pix/boleto/transferência |
| `DebitCard` | `debit_card` | Cartão de débito ligado a ativo |
| `CreditCard` | `credit_card` | Instrumento que gera dívida |
| `Other` | `other` | Meio com handler explicitamente suportado |

## Regras

Nome específico do método é catálogo pessoal. Família seleciona comportamento permitido e não torna um meio externo automaticamente implementado. Moeda e origem são verificadas no pagamento.

## Persistência e implementação

Usar em `payment_methods` com cast Eloquent e validação/constraints. Aplicar no fluxo de [domínio](doc:financas); a presença de um case não habilita automaticamente uma funcionalidade.

- [ ] Confirmar regras e transições com exemplos.
- [ ] Criar enum, rótulos e cast na implementação de 2027.
- [ ] Validar valores inválidos, escopo e conversão de datas/valores aplicável.
