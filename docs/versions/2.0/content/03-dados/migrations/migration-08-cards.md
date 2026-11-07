---
id: migration-08-cards
title: Migration 08 — Cartões, ciclos e parcelas
description: Contrato lógico de tabelas, FKs, índices e conversão: cartões, ciclos e parcelas.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, cartoes, componentes-tecnicos
source_refs:
updated: 2026-11-07
---

## `credit_cards`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, issuer_party_id, name, last_four` | Identidade, emissor opcional e identificador visual limitado. |
| `liability_ledger_account_id, currency, limit_minor` | Passivo interno único e limite exato. |
| `default_payment_account_id, archived_at` | Conta sugerida, sem obrigar todos os pagamentos a sair dela. |

## `card_rule_versions`

| Campo ou grupo | Contrato |
| --- | --- |
| `card_id, effective_from, closing_day, due_day` | Regras versionadas; dias 1–31 com ajuste para fim de mês. |
| `cycle_timezone, closing_inclusive` | Fuso e convenção de corte explícitos. |

## `credit_card_invoices`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, card_id, cycle_key, reference_month, rule_version_id` | Competência normalizada e versão da regra. |
| `closing_on, due_on, cycle_state` | Datas históricas congeladas e InvoiceCycleState. |
| `closed_total_minor, closed_at, revision` | Snapshot de fechamento e revisão; quitação é calculada. |

## `card_charges`

| Campo ou grupo | Contrato |
| --- | --- |
| `card_id, payment_id, amount_minor` | Operação no instrumento de crédito e funding da compra/devolução, valor positivo. |
| `debt_direction, reverses_charge_id` | CardDebtDirection informa aumento/redução de dívida; devolução aponta para o débito original. |
| `installment_count, purchased_on` | Total de parcelas e data de compra; não duplicar a despesa a cada ciclo. |

## `card_installments`

| Campo ou grupo | Contrato |
| --- | --- |
| `charge_id, invoice_id, sequence, amount_minor` | Parcelas positivas, ordem única por charge; fatura futura pode ser resolvida depois. |
| `expected_due_on, adjusted_from_installment_id` | Agenda e vínculo de ajuste/estorno quando necessário. |
| `sinal herdado` | Valor positivo; efeito vem de debt_direction do charge, sem confundir direção da dívida com lado contábil do posting. |

## `invoice_payment_allocations`

| Campo ou grupo | Contrato |
| --- | --- |
| `payment_id, invoice_id, amount_minor` | Vários pagamentos por fatura e várias faturas por operação autorizada; valor positivo. |

## Integridade, índices e comportamento

- UNIQUE(space_id, card_id, cycle_key) e UNIQUE(charge_id, sequence). reference_month é referência de apresentação, permitindo ciclos excepcionais distintos no mesmo mês. Soma das parcelas igual ao débito financiado.
- Pagamento de fatura debita passivo e credita ativo; não reproduz a compra como despesa.
- Créditos de devolução têm valor e ciclo explícitos; devolução parcial não cancela toda a dívida nem reabre cobrança da compra.
- Fechamento e liquidação são dimensões distintas. Atraso depende de vencimento e saldo. Crédito excedente tem destino próprio ou é recusado; nunca desaparecer o excesso.
- Arquivar permite pagar dívida/receber estorno histórico, mas impede novas compras usuais. Recalcular limite considera parcelas futuras e pagamentos conforme critério documentado.
- Financiamento de fatura, adicionais e limite conjunto entre instrumentos são etapas futuras; não criar campos ambíguos que simulam esses produtos.

## Funding parcial e unicidade

UNIQUE(space_id, payment_id) em card_charges garante um cartão por pagamento. Seu amount_minor corresponde ao pagamento financiado e à soma de suas alocações. Uma compra com dois cartões usa dois pagamentos e dois charges. A soma das parcelas corresponde somente ao valor financiado por cada charge.

Arquivamento conserva parcelas futuras e permite pagar faturas e processar correções vinculadas. O fluxo de entrada histórica valida purchased_on anterior ao arquivamento e registra a exceção.

## Alocações inversas de pagamento de fatura

invoice_payment_allocations acrescenta original_allocation_id nullable para correções. A operação inversa referencia o pagamento original e as alocações afetadas, preservando fatura, moeda e limites cumulativos. A projeção de quitação soma pagamentos efetivos e suas correções em cada fatura.

## Competência e limite

cycle_key é estável durante alterações de regra e importação. No ciclo mensal usual pode derivar da competência, e ciclos excepcionais recebem identidade própria confirmada. Uma compra guarda purchased_on e pode guardar processed_on quando informado. O usuário pode ajustar a fatura real com auditoria e revisão das parcelas afetadas.

O protótipo inicial de limite considera principal ainda devido, incluindo parcelas futuras, pagamentos efetivos e créditos aplicáveis. A UI identifica data/base da informação e separa saldo de faturas vencidas e comprometimento futuro. Registrar compra histórica que supera o limite configurado apresenta aviso e preserva o fato confirmado.

## Conversão do James 1.x

Converter amount_paid/payment_transaction_id em histórico disponível, sem inventar parcelas de pagamentos já agregados. Compra legado efetivada em conta ao quitar fatura não pode gerar nova saída de caixa na conversão. Preservar closing_date/due_date reais.

## Componentes previstos

ResolveInvoiceCycle; CloseInvoice; PayInvoice; ReverseInvoicePayment; PlanInstallments; UsedCreditQuery; reconciliador de saldos legados.
