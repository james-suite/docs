---
id: enum-payment-payer-type
title: Enum — PaymentPayerType
description: Identidade de quem pagou; o vínculo de conta determina se há movimento pessoal.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas, acertos
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Self` | `self` | O usuário foi quem pagou; se houve movimentação em conta acompanhada, o instrumento pertence ao próprio usuário. |
| `Contact` | `contact` | Um contato pagou; exige `payer_party_id`. |
| `Unknown` | `unknown` | Pagamento ou liquidação efetiva foi registrada, mas o pagador não foi confirmado. |

## Regras e transições

O pagador não determina quem consumiu, quem é dono do veículo nem se existe dívida entre pessoas. `Self` exige a conta de origem quando houve saída de saldo pessoal. `Contact` não aceita uma conta privada do contato nem debita uma conta do usuário. Criar ou liquidar uma obrigação exige vínculo explícito a um acerto.

Uma nota fiscal não escolhe o pagador: a interface pergunta ao revisar a captura. Vários pagamentos do mesmo documento podem ter pagadores e datas diferentes.
