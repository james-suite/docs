---
id: migration-07-payments
title: Migration 07 — Pagamentos, alocações e saldo
description: Contrato lógico de tabelas, FKs, índices e conversão: pagamentos, alocações e saldo.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, financas, veiculos, componentes-tecnicos, enum-payment-payer-type
source_refs:
updated: 2026-10-26
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 05, 06. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `financial_payments`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, kind, status` | Identidade, PaymentKind e PaymentStatus. |
| `amount_minor, currency, scheduled_on` | Valor positivo, moeda e data prevista. |
| `effective_on, posted_at, time_precision, observed_timezone` | Data financeira local obrigatória na efetivação; instante opcional quando conhecido, TimePrecision e fuso de origem. Não inventar horário em registros legados. |
| `created_at` | Instante real de registro/importação; não substitui a data financeira. |
| `payer_type, payer_party_id` | Pagador usuário, contato ou desconhecido; contato obrigatório somente para o tipo Contact. |
| `source_ledger_account_id, destination_ledger_account_id` | Instrumentos/contrapartidas autorizados conforme tipo; origem pessoal obrigatória só quando o usuário pagou e houve movimento na própria conta. |
| `payment_method_id, reverses_payment_id, refunds_payment_id` | Método; referência à correção técnica ou à devolução comercial, com exclusividade conforme PaymentKind. |
| `idempotency_key, reason` | Chave única por espaço; motivo obrigatório para ajustes/estornos. |

## `financial_payment_allocations`

| Campo ou grupo | Contrato |
| --- | --- |
| `payment_id, transaction_id, amount_minor` | N:N entre pagamentos e transações; valor positivo; mesma moeda. |
| `original_allocation_id` | FK para alocação afetada por reversão/devolução parcial; validar espaço, moeda e propósito. |
| `UNIQUE(payment_id, transaction_id)` | Uma alocação agregada por par; granularidade por item só se requisito exigir. |

## `ledger_journals`

| Campo ou grupo | Contrato |
| --- | --- |
| `payment_id, currency, effective_on, posted_at` | Lote interno com payment_id único; data efetiva e instante opcional herdados da operação. |
| `reverses_journal_id, correlation_id` | Referência à reversão e correlação da operação. |

## `ledger_postings`

| Campo ou grupo | Contrato |
| --- | --- |
| `journal_id, ledger_account_id, side, amount_minor` | EntrySide e valor positivo; mesma moeda/espaço do lote. |
| `description` | Contexto curto; o documento de negócio é a fonte do detalhe. |

## Integridade, índices e comportamento

- Livro interno condicionado ao gate de arquitetura. Lote balanceado por moeda, sem postagem parcial; conferir soma sob bloqueio/constraint trigger ou procedimento transacional revisado.
- Antes de efetivar, bloquear documentos/alocações e revalidar saldo a pagar. A soma alocada em pagamento do tipo transaction coincide com seu valor. Transferências e pagamentos de fatura usam regras próprias.
- Reversão técnica desfaz um pagamento incorreto e pode reabrir o valor a pagar. Devolução comercial gera documento relacionado e novo fluxo inverso, mantendo a compra original paga e reduzindo a despesa líquida. Não usar o mesmo tratamento para os dois casos.
- Cada reversão/devolução parcial identifica a alocação original. A soma cumulativa de retornos não excede cada alocação nem o pagamento; validar sob bloqueio, inclusive quando um pagamento quitou vários documentos.
- Pagamento pode quitar vários documentos e documento pode ter vários pagamentos. Não misturar valores de moedas diferentes sem operação de câmbio própria.
- Pagamento efetivo por contato quita o documento sem fabricar débito em conta pessoal. Atribuição de pagador não cria por si só dívida, crédito, copropriedade ou rateio; obrigações exigem vínculo explícito a acerto.
- Tipo Self exige origem pessoal quando o movimento sai da conta do usuário; Contact exige `payer_party_id` e não aponta para conta privada do contato; Unknown mantém a incerteza e não pode gerar lançamento bancário pessoal presumido.
- Postings efetivados são imutáveis. Soft delete de rascunho não é mecanismo para retirar pagamento do saldo; reversão gera lote inverso. Essa exceção à convenção genérica de SoftDeletes precisa de ADR 2.0.
- Índices UNIQUE(space_id, idempotency_key), por conta/data e por transaction_id. Requisição repetida com mesma chave e payload diferente é rejeitada.

## Conversão do James 1.x

Cada transação posted é analisada conforme instrumento. Transfer_pair_id vira uma transferência única. Pagamentos agregados de faturas exigem conversão específica no pacote 08.

## Componentes previstos

PostPayment; ReversePayment; AllocatePayment; TransferFunds; JournalBuilder; lockForUpdate; idempotência no banco; validação de pagador/instrumento.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
