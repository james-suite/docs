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
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Transaction` | `transaction` | Pagamento ou recebimento de documentos |
| `Transfer` | `transfer` | Transferência entre contas |
| `InvoicePayment` | `invoice_payment` | Pagamento de fatura |
| `Refund` | `refund` | Devolução comercial, com documento vinculado |
| `Reversal` | `reversal` | Correção técnica que desfaz uma operação anterior |
| `OpeningBalance` | `opening_balance` | Saldo inicial |
| `Adjustment` | `adjustment` | Ajuste explícito |

## Regras e transições

Reversal desfaz uma operação incorreta e pode reabrir obrigação. Refund registra devolução comercial, preserva a compra original paga e ajusta o resultado líquido. Ambos apontam para alocações originais e têm limites cumulativos. Cada tipo determina contrapartidas e alocações válidas. Pagamento de fatura não gera despesa; ajuste precisa de justificativa.

## Persistência

Aplicar em `financial_payments` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:financas) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
