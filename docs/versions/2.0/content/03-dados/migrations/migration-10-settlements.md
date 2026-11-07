---
id: migration-10-settlements
title: Migration 10 — Rateios e liquidações de acertos
description: Contrato lógico de tabelas, FKs, índices e conversão: rateios e liquidações de acertos.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, acertos, documentos-arquivos, migration-22-documents, componentes-tecnicos, enum-payment-payer-type
source_refs:
updated: 2026-11-07
---

## `settlement_groups`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, source_transaction_id, description, occurred_on, total_minor, currency, mode` | Rateio de despesa; transação de origem opcional e única quando existir, e nesse caso deve ser Expense. Valores/moeda devem coincidir com a fonte; grupo organizado sem transação continua válido. |

## `split_participants`

| Campo ou grupo | Contrato |
| --- | --- |
| `group_id, party_id, is_self` | Participante do rateio; XOR entre `party_id` e `is_self`. `party_id` não se repete no grupo e pode haver no máximo um participante `Self`; a pessoa organizadora não precisa constar. |
| `display_snapshot` | Nome no momento da operação, sem perder referência atual. |

## `split_shares`

| Campo ou grupo | Contrato |
| --- | --- |
| `participant_id, amount_minor, weight` | Cota opcional para cada participante consumidor, com valor calculado e peso opcional conforme o modo do grupo. Minha cota pode ser zero; participante sem parcela própria não é criado como contato fictício. |

## `split_contributions`

| Campo ou grupo | Contrato |
| --- | --- |
| `participant_id, amount_minor, paid_on, financial_payment_allocation_id` | Quem pagou ao fornecedor; FK opcional para a alocação financeira efetiva do pagador confirmado, com mesmo valor/data. UNIQUE na alocação não nula impede usar a mesma quantia em dois rateios. |

## `split_settlement_edges`

| Campo ou grupo | Contrato |
| --- | --- |
| `group_id, plan_revision, debtor_participant_id, creditor_participant_id, amount_minor` | Plano explícito de compensação entre saldos de participantes; valor positivo e participantes distintos do mesmo grupo. |
| `UNIQUE(group_id, plan_revision, debtor_participant_id, creditor_participant_id)` | Evita obrigações duplicadas para a mesma aresta; plano fecha os saldos das cotas financiadas e contribuições, preservando o saldo devido ao fornecedor. |

## `settlement_obligations`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, party_id, direction, amount_minor, currency` | Obrigação individual de receber/pagar e sua contraparte `Party`; essa relação é a dimensão de contato/empresa em relatórios, sem tag artificial. |
| `group_id, source_edge_id, source_transaction_id, occurred_on, due_on, archived_at` | Origem por aresta de rateio ou obrigação direta; `source_transaction_id` só pode aparecer fora de grupo e é opcional. `source_edge_id` único materializa somente arestas que envolvem o usuário. |

## `settlement_payments / settlement_payment_allocations`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, paid_on, amount_minor, currency, payer_type, payer_party_id, reverses_settlement_payment_id` | Evento efetivo de liquidação. `payer_type` usa PaymentPayerType: `Self`, `Contact` ou `Unknown`; `payer_party_id` é obrigatório somente para `Contact`, eliminando a ambiguidade entre usuário pagador e pagador externo desconhecido. |
| `method, financial_payment_id` | PaymentMethod nullable se não houver pagamento financeiro vinculado. `financial_payment_id` é nullable e único: associa, quando o fluxo de conta pessoal for acompanhado, um movimento FinancialPayment do tipo Settlement na liquidação ou Reversal em sua correção inversa. |
| `settlement_payment_id, obligation_id, amount_minor, original_allocation_id` | Alocação da liquidação a uma ou mais obrigações de mesma direção/moeda; soma igual ao valor liquidado e nunca superior ao saldo aberto. Contrapartes podem diferir somente quando o pagador/beneficiário e as obrigações forem associados explicitamente. |

## Integração com pagamentos financeiros

O pagamento original ao fornecedor fica em `financial_payments` com alocação à transação. Uma liquidação entre pessoas não altera essa alocação. Se uma liquidação movimentar conta pessoal, o SettlementPayment referencia um FinancialPayment do tipo Settlement por financial_payment_id; não cria linha em `financial_payment_allocations`. Seu lançamento contábil reduz caixa e a conta de receber/pagar, sem tocar em receita, despesa ou quitação da compra original. O `payer_party_id` registra pagador externo quando confirmado; as obrigações alocadas registram quem devia ou a quem se devia. Se os papéis divergirem, não inferir que o pagador é o titular de todas as obrigações quitadas. Se o pagamento financeiro estiver ligado, valor, moeda, data efetiva e identidade de quem pagou precisam reconciliar com o evento de acerto.

## Integridade, índices e comportamento

- Organizador pode não consumir e pode não pagar. Igual divide somente pelos consumidores selecionados. Não criar automaticamente uma parcela minha.
- Participante, cota e contribuição são registros distintos: alguém pode consumir sem pagar ou pagar sem consumir. Cada Party aparece uma vez no grupo, `Self` é opcional e único, cada participante tem no máximo uma cota e as cotas somam o total do grupo.
- Se o grupo aponta para uma transação, exigir que ela esteja confirmada, seja uma despesa e pertença ao mesmo espaço; a transação continua sendo a fonte do valor, moeda e pagamentos ao fornecedor.
- Soma das shares igual ao total; contribuições registram o que já foi pago. Relações entre contatos servem ao rateio privado, sem vincular contas de usuários reais.
- Liquidação de obrigação não vira receita/despesa pessoal. Devolução comercial do fornecedor continua no fluxo Refund do pacote 07; ela é diferente de dinheiro devolvido por um contato.
- Cotas, contribuições e suas alocações de financiamento são a fonte do plano sobre valores já financiados. Obrigações pessoais são materializadas de arestas únicas envolvendo Eu. Arestas somente entre contatos são informativas e não entram no saldo pessoal.
- Cada obrigação de grupo corresponde a uma única aresta que envolve `Self`; a Party e a direção (`Receivable`/`Payable`) devem corresponder ao participante oposto. Obrigação direta não tem `group_id` nem `source_edge_id` e pode apontar para a transação de origem; obrigação originada de rateio usa grupo/aresta e não repete a transação nessa coluna.
- Registrar contribuição classifica um pagamento existente; criar obrigação não gera nova saída de caixa. Validar valor e vínculo da alocação para não duplicar integração. Grupo só organizado, sem fluxo próprio, não cria obrigação pessoal nem ledger.
- Saldo por contraparte é calculado pelas obrigações menos alocações de liquidação; consumo por contraparte vem das shares. Exibir esses dados separados e sem reclassificar a contraparte como tag.
- Uma liquidação pode alocar a várias obrigações na mesma direção e moeda. Se o pagador efetivo diferir das pessoas devedoras, registrar o tipo/identidade do pagador e as obrigações quitadas explicitamente. Não presumir que quem enviou o dinheiro é titular de todas as dívidas liquidadas.
- `SettlementPayment` representa liquidação efetiva; previsão de pagamento fica na obrigação, vencimento, tarefa ou notificação. Se houver `financial_payment_id`, data, valor, moeda, meio e pagador precisam reconciliar; o financeiro é fonte do método quando ligado.
- Se `financial_payment_id` estiver preenchido, tipo e identidade do pagador, data, valor, moeda e meio precisam corresponder ao `FinancialPayment`; `Contact` exige a mesma Party e `Self` exige a conta pessoal compatível com o sentido do fluxo. `Unknown` não pode produzir posting pessoal presumido.
- Se já houver liquidações, alterar o rateio exige ajuste versionado, sem apagar/recriar obrigações quitadas. Liquidação parcial bloqueia obrigações e revalida saldo; estorno preserva lançamento original.

## Compra parcialmente paga e financiamento das cotas

split_funding_allocations guarda id, space_id, contribution_id, share_id e amount_minor positivo. Contribuição e cota precisam ser do mesmo grupo. UNIQUE(contribution_id, share_id) agrega o par. Cada contribuição confirmada é integralmente atribuída, e a soma financiada de uma cota fica limitada ao seu valor de consumo. Alocações são criadas/revisadas sob bloqueio das contribuições e cotas.

Cotas descrevem todo o consumo, enquanto a matriz de financiamento descreve quem já adiantou cada parcela. O saldo do fornecedor vem da transação e seus pagamentos. O plano de compensação considera contribuição menos consumo já financiado, e cria obrigações entre pessoas somente sobre essa parte. Propor uma distribuição pode usar cotas e centavos remanescentes com desempate estável, apresentando a prévia para confirmação.

Exemplo de compra de R$ 120 para Ana e Bruno, R$ 60 por pessoa. Eu pago R$ 60 ao fornecedor e atribuo R$ 30 a cada cota. As obrigações são R$ 30 de Ana e R$ 30 de Bruno para mim, e a transação mantém R$ 60 pendentes ao fornecedor. Se o rateio estiver só planejado e sem pagamentos, as cotas existem com financiamento zero.

Cada financiamento gera arestas com origem estável, revisão e chave idempotente. Novas contribuições acrescentam financiamento. Uma obrigação já liquidada conserva origem, contraparte e alocações. Corrigir consumo ou contribuição após liquidação exige prévia de operações de ajuste e reversão. A interface pode manter o grupo como rascunho até reconciliar a matriz.

## Reversão de liquidação

reverses_settlement_payment_id aponta à liquidação original. Cada alocação inversa referencia original_allocation_id, conserva a obrigação/moeda e devolve ao saldo aberto o valor corrigido. A soma das reversões por alocação fica limitada ao efetivamente liquidado. O registro original permanece consultável.

Quando houver FinancialPayment vinculado, a mesma operação cria o Reversal financeiro no instrumento original e a reversão das alocações de acerto. Revalidar tudo numa transação com idempotência e ordem estável de bloqueio. Uma liquidação externa usa a mesma correção das obrigações, com seu motivo e autoria.

## Conversão do James 1.x

TheyOwe/IOwe e TheyPaid/IPaid orientam a proposta inicial de obrigações e liquidações. A preparação permite reconstruir rateios, contribuições e financiamento conforme o modelo do 2.0, com revisão de saldos, arquivamento e decisões de correção. Reconciliação considera a antiga regra de saldo diário limitado a zero; diferença exige relatório.

## Componentes previstos

SaveSplit; CalculateShares; RecordContribution; SettleObligations; ReverseSettlement; ArchiveSettlement.
