---
id: enum-payment-payer-type
title: Enum — PaymentPayerType
description: Quem pagou uma operação e se o pagamento deve movimentar uma conta pessoal.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas, acertos
source_refs:
updated: 2026-10-26
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Self` | `self` | O usuário pagou com dinheiro de um instrumento pessoal. |
| `Contact` | `contact` | Um contato pagou; exige `payer_party_id`. |
| `Unknown` | `unknown` | O documento está quitado ou a informação foi registrada, mas o pagador não foi confirmado. |

## Regras e transições

O pagador não determina quem consumiu, quem é dono do veículo nem se existe dívida entre pessoas. `Self` exige a conta de origem quando houve saída de saldo pessoal. `Contact` não aceita uma conta privada do contato nem debita uma conta do usuário. Criar obrigação ou reembolso exige vínculo explícito a um acerto.

Uma nota fiscal não escolhe o pagador: a interface pergunta ao revisar a captura. Vários pagamentos do mesmo documento podem ter pagadores e datas diferentes.

## Persistência

Aplicar em `financial_payments` conforme [Migration 07](doc:migration-07-payments). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco.

## Checklist

- [ ] Revisar os cases no fluxo de [finanças](doc:financas) e [acertos](doc:acertos).
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger coerência entre pagador, Party e instrumento de origem.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
