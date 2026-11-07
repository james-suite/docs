---
id: enum-payment-method
title: Enum — PaymentMethod
description: Detalhe opcional do meio usado em um pagamento.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: enums, migration-05-accounts, migration-07-payments, financas
source_refs:
updated: 2026-11-07
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Cash` | `cash` | Dinheiro físico |
| `Pix` | `pix` | Pix |
| `DebitCard` | `debit_card` | Débito na conta escolhida |
| `CreditCard` | `credit_card` | Crédito no cartão identificado |
| `BankSlip` | `bank_slip` | Boleto pago pela conta escolhida |
| `BankTransfer` | `bank_transfer` | Transferência bancária |
| `Other` | `other` | Outro meio explicitamente informado |

## Contrato de uso

Campo nullable em financial_payments e em liquidações externas sem FinancialPayment. A interface pode inferir crédito a partir do cartão selecionado e dinheiro a partir da conta Wallet, com origem dessa inferência identificável. Para conta corrente, Pix/débito/boleto são detalhes opcionais. Dados legados sem informação preservam null.

A identidade do instrumento controla o efeito sobre saldo e dívida. Validar o enum informado contra esse instrumento e manter os códigos exportáveis. O planejamento usa esse enum diretamente, dispensando tabela e cadastro de meios por conta.
