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
updated: 2026-11-07
---

## `financial_payments`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, kind, status` | Identidade, PaymentKind e PaymentStatus. |
| `amount_minor, currency, scheduled_on` | Valor positivo, moeda e data prevista. |
| `effective_on, posted_at, time_precision, observed_timezone` | Data financeira local obrigatória na efetivação; instante opcional quando conhecido, TimePrecision e fuso de origem. Não inventar horário em registros legados. |
| `created_at` | Instante real de registro/importação; não substitui a data financeira. |
| `payer_type, payer_party_id` | Pagador usuário, contato ou desconhecido; contato obrigatório somente para o tipo Contact. |
| `source_account_id, destination_account_id` | FKs opcionais para FinancialAccount conforme direção/tipo da operação. Transferência identifica ambas. O servidor deriva as contas do ledger dos instrumentos confirmados. Crédito identifica o cartão pelo CardCharge ligado ao pagamento. |
| `method, reverses_payment_id` | PaymentMethod nullable e referência ao pagamento tecnicamente revertido. Devoluções referenciam os pagamentos originais pelas alocações. Settlement recebe vínculo à liquidação do pacote 10. |
| `idempotency_key, reason` | Chave única por espaço; motivo obrigatório para ajustes/estornos. |

## `financial_payment_allocations`

| Campo ou grupo | Contrato |
| --- | --- |
| `payment_id, transaction_id, amount_minor` | N:N entre pagamentos e transações; valor positivo; mesma moeda. |
| `original_allocation_id` | FK para alocação afetada por reversão/devolução parcial; validar espaço, moeda e propósito. |
| Unicidade por tipo de alocação | Pagamento usual tem uma alocação por par payment/transaction com original_allocation_id nulo. Correções/devoluções permitem uma por payment/transaction/original_allocation_id, garantindo múltiplas origens sem repetição do mesmo vínculo. |

`PaymentKind::Settlement` não usa esta tabela: a liquidação é alocada às obrigações em `settlement_payment_allocations` e ligada ao movimento de caixa por FK nullable no pacote 10. Assim, receber R$ 40 de um contato reduz o crédito a receber sem aparecer como renda nem como nova alocação da compra.

## `ledger_journals`

| Campo ou grupo | Contrato |
| --- | --- |
| `payment_id, currency, effective_on, posted_at` | Lote interno com payment_id nullable e único quando informado. Data efetiva e instante opcional vêm da operação de origem. |
| `source_record_id, source_revision, operation_key` | Origem e chave única por espaço para lançamentos locais, inclusive reclassificação de acerto sem novo pagamento. |
| `reverses_journal_id, correlation_id` | Referência à reversão e correlação da operação. |

O vínculo entre pagamento e journal é opcional quando o pagamento quita um documento, mas não movimenta conta pessoal acompanhada nem cria obrigação pessoal. Pagamento efetivo por contato pode liquidar a transação sem gerar posting pessoal; quando houver movimento de conta ou liquidação de obrigação acompanhada, a operação financeira gera seu journal balanceado.

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
- Settlement ligado a conta pessoal gera posting contra ativo/passivo de acerto, não contra receita/despesa. Para recebimento, registrar `payer_type=Contact` e conta de destino pessoal; para pagamento, `payer_type=Self` e conta de origem. `payer_party_id` identifica quem enviou o dinheiro e pode diferir da Party das obrigações quitadas, que fica explícita nas alocações do acerto. Um SettlementPayment sem `financial_payment_id` pode representar acerto efetivo fora das contas acompanhadas, sem posting inventado.
- Tipo Self exige origem pessoal quando o movimento sai da conta do usuário; Contact exige `payer_party_id` e não aponta para conta privada do contato; Unknown mantém a incerteza e não pode gerar lançamento bancário pessoal presumido.
- Postings efetivados são imutáveis. Soft delete de rascunho não é mecanismo para retirar pagamento do saldo; reversão gera lote inverso. Essa exceção à convenção genérica de SoftDeletes precisa de ADR 2.0.
- Índices UNIQUE(space_id, idempotency_key), por conta/data e por transaction_id. Requisição repetida com mesma chave e payload diferente é rejeitada.

## Instrumentos e pagamentos mistos

Cada linha de pagamento representa um valor, pagador, data e instrumento. Uma compra pode combinar dinheiro físico, Pix e crédito, inclusive dois cartões ou duas contas. Vários métodos na mesma compra produzem vários pagamentos alocados ao mesmo transaction_id.

Para compra paga por Self, validar uma origem de fundos por pagamento, conta financeira ou CardCharge. Dinheiro físico usa uma FinancialAccount do tipo Wallet. Débito e Pix usam a conta de ativo escolhida. Um crédito de R$ 180 produz CardCharge de R$ 180, ainda que a compra inteira seja R$ 300. UNIQUE(space_id, payment_id) em card_charges garante um instrumento de crédito por linha de pagamento.

A soma de alocações efetivas indica o que já foi pago ao fornecedor. Itens e ajustes globais descrevem o valor econômico da compra. O método opcional não escolhe conta, cartão ou handler de lançamentos. Reversão parcial se liga à alocação e ao instrumento originais.

A alocação monetária permanece entre pagamento e transação. Uma eventual visualização de caixa por categoria de item deve declarar sua regra analítica de distribuição, cobertura e arredondamento. Preservar como estimativa qualquer rateio proporcional usado nessa consulta.

## Reversões e devoluções no cálculo

A consulta do ledger soma o original e seus lançamentos inversos. O status Reversed funciona como projeção de apresentação e acompanha as reversões. Uma consulta que usa os lançamentos inversos conserva também o original no conjunto de cálculo.

Para Refund, transaction_id da alocação identifica a transação de devolução e original_allocation_id identifica a alocação da compra. Validar related_transaction_id, moeda, contraparte, valor remanescente e instrumento. A transação de origem continua quitada. A soma de devoluções e correções respeita a exposição original com suas reversões já realizadas.

Para reversão de pagamento de fatura, invoice_payment_allocations também guarda original_allocation_id. Sua alocação inversa conserva a fatura original e obedece ao limite efetivamente pago. A correção de uma liquidação usa ainda a reversão de SettlementPayment e suas alocações do pacote 10.

## Devolução de compra paga em vários instrumentos

Uma devolução pode referenciar várias alocações originais. Compra de R$ 100 paga por Pix de R$ 60 e cartão de R$ 40 pode ser devolvida integralmente por Pix em um único pagamento de R$ 100. Duas alocações ao documento de devolução apontam respectivamente às origens de R$ 60 e R$ 40.

O destino efetivo da devolução é confirmado pela pessoa. Se o fornecedor devolver na conta, o saldo bancário aumenta e a dívida original no cartão permanece até sua quitação. Devolução no cartão cria redução de dívida no cartão identificado. Correção técnica conserva o instrumento da operação corrigida.

Para Transfer, OpeningBalance e Adjustment, payer_type pode ser nulo quando a operação representa movimentação interna ou ajuste técnico. Conta de origem/destino e motivo determinam a direção do saldo. Saldo inicial negativo usa valor positivo com direção de saída e contrapartida patrimonial, preservando a representação legada no mapa.

## Rateio definido após o pagamento

Um rateio criado depois de uma compra paga pode exigir reclassificar o que já saiu do caixa como consumo próprio ou valor adiantado. O journal dessa revisão referencia o grupo/obrigação e sua operação, com payment_id nulo. O lançamento transfere a contrapartida entre despesa e receber/pagar, mantendo o saldo da conta.

Exemplo do protótipo com pagamento de R$ 120 já lançado como despesa. Ao confirmar que todo o consumo era de contatos e criar R$ 120 a receber, reclassificar R$ 120 da contrapartida de despesa para o ativo de acerto. A posterior liquidação reduz esse ativo e aumenta caixa. Consultas de consumo pessoal usam as cotas confirmadas e os relatórios reconciliam as contrapartidas.

O gate do ledger deve provar esse fluxo, o caso pago por contato e a correção de uma obrigação sem movimentação financeira. Chaves de operação impedem gerar a mesma reclassificação em duas gravações.

## Dinheiro entregue e troco

cash_tendered_minor e change_minor são opcionais para pagamento em espécie com dados conhecidos. amount_minor representa a quantia líquida aplicada à compra e coincide com entregue menos troco quando ambos estiverem informados. Em compra de R$ 150 com R$ 200 entregues e R$ 50 de troco, a conta Dinheiro físico diminui R$ 150.

Dados de pagamento declarados na nota são candidatos para revisão. O leitor preserva valor entregue, troco e total líquido em seus campos de origem. A identidade de quem pagou e o instrumento pessoal seguem confirmados pelo fluxo financeiro.

## Conversão do James 1.x

Cada transação posted é analisada conforme instrumento. Transfer_pair_id vira uma transferência única. Pagamentos agregados de faturas exigem conversão específica no pacote 08.

## Componentes previstos

PostPayment; ReversePayment; AllocatePayment; TransferFunds; JournalBuilder; lockForUpdate; idempotência no banco; validação de pagador/instrumento.
