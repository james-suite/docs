---
id: enum-payment-kind
title: Enum — PaymentKind
description: Valores e invariantes propostos de PaymentKind.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Transaction` | `transaction` | Pagamento ou recebimento de documentos |
| `Transfer` | `transfer` | Transferência entre contas |
| `InvoicePayment` | `invoice_payment` | Pagamento de fatura |
| `Settlement` | `settlement` | Movimento de caixa ligado à liquidação de obrigação entre o usuário e uma contraparte |
| `Refund` | `refund` | Devolução comercial, com documento vinculado |
| `Reversal` | `reversal` | Correção técnica que desfaz uma operação anterior |
| `OpeningBalance` | `opening_balance` | Saldo inicial |
| `Adjustment` | `adjustment` | Ajuste explícito |

## Regras e transições

Settlement só é criado junto de um `SettlementPayment`; não recebe alocação de transação e sua contrapartida é uma obrigação a receber/pagar, não receita/despesa. Reversal desfaz uma operação incorreta e pode reabrir obrigação. Refund registra devolução comercial, preserva a compra original paga e ajusta o resultado líquido. Reversal e Refund apontam para alocações originais e têm limites cumulativos. Cada tipo determina contrapartidas e alocações válidas. Pagamento de fatura não gera despesa; ajuste precisa de justificativa.
