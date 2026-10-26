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
updated: 2026-10-25
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 05, 07. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

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
| `record_id, card_id, reference_month, rule_version_id` | Competência normalizada e versão da regra. |
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

- UNIQUE(space_id, card_id, reference_month); UNIQUE(charge_id, sequence). Soma das parcelas igual ao débito financiado.
- Pagamento de fatura debita passivo e credita ativo; não reproduz a compra como despesa.
- Créditos de devolução têm valor e ciclo explícitos; devolução parcial não cancela toda a dívida nem reabre cobrança da compra.
- Fechamento e liquidação são dimensões distintas. Atraso depende de vencimento e saldo. Crédito excedente tem destino próprio ou é recusado; nunca desaparecer o excesso.
- Arquivar permite pagar dívida/receber estorno histórico, mas impede novas compras usuais. Recalcular limite considera parcelas futuras e pagamentos conforme critério documentado.
- Financiamento de fatura, adicionais e limite conjunto entre instrumentos são etapas futuras; não criar campos ambíguos que simulam esses produtos.

## Conversão do James 1.x

Converter amount_paid/payment_transaction_id em histórico disponível, sem inventar parcelas de pagamentos já agregados. Compra legado efetivada em conta ao quitar fatura não pode gerar nova saída de caixa na conversão. Preservar closing_date/due_date reais.

## Componentes previstos

ResolveInvoiceCycle; CloseInvoice; PayInvoice; ReverseInvoicePayment; PlanInstallments; UsedCreditQuery; reconciliador de saldos legados.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
