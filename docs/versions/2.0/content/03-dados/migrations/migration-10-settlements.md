---
id: migration-10-settlements
title: Migration 10 — Rateios e liquidações de acertos
description: Contrato lógico de tabelas, FKs, índices e conversão: rateios e liquidações de acertos.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, acertos, componentes-tecnicos, enum-payment-payer-type
source_refs:
updated: 2026-10-31
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 03, 06, 07. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

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
| `participant_id, amount_minor, paid_on, financial_payment_allocation_id` | Quem pagou ao fornecedor; FK opcional para a alocação financeira efetiva do usuário. UNIQUE na alocação não nula impede usar a mesma quantia em dois rateios. |

## `split_settlement_edges`

| Campo ou grupo | Contrato |
| --- | --- |
| `group_id, plan_revision, debtor_participant_id, creditor_participant_id, amount_minor` | Plano explícito de compensação entre saldos de participantes; valor positivo e participantes distintos do mesmo grupo. |
| `UNIQUE(group_id, plan_revision, debtor_participant_id, creditor_participant_id)` | Evita obrigações duplicadas para a mesma aresta; plano fecha os saldos calculados de cotas e contribuições. |

## `settlement_obligations`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, party_id, direction, amount_minor, currency` | Obrigação individual de receber/pagar e sua contraparte `Party`; essa relação é a dimensão de contato/empresa em relatórios, sem tag artificial. |
| `group_id, source_edge_id, source_transaction_id, occurred_on, due_on, archived_at` | Origem por aresta de rateio ou obrigação direta; `source_transaction_id` só pode aparecer fora de grupo e é opcional. `source_edge_id` único materializa somente arestas que envolvem o usuário. |

## `settlement_payments / settlement_payment_allocations`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, paid_on, amount_minor, currency, payer_type, payer_party_id` | Evento efetivo de liquidação. `payer_type` usa PaymentPayerType: `Self`, `Contact` ou `Unknown`; `payer_party_id` é obrigatório somente para `Contact`, eliminando a ambiguidade entre usuário pagador e pagador externo desconhecido. |
| `payment_method_id, financial_payment_id` | Meio opcional se não houver pagamento financeiro vinculado. `financial_payment_id` é nullable e único: associa, quando o fluxo de conta pessoal for acompanhado, um movimento FinancialPayment do tipo Settlement. |
| `settlement_payment_id, obligation_id, amount_minor` | Alocação da liquidação a uma ou mais obrigações de mesma direção/moeda; soma igual ao valor liquidado e nunca superior ao saldo aberto. Contrapartes podem diferir somente quando o pagador/beneficiário e as obrigações forem associados explicitamente. |

## Integração com pagamentos financeiros

O pagamento original ao fornecedor fica em `financial_payments` com alocação à transação. Uma liquidação entre pessoas não altera essa alocação. Se uma liquidação movimentar conta pessoal, `financial_payments.kind=settlement` aponta ao `settlement_payment` por `financial_payment_id`; não cria linha em `financial_payment_allocations`. Seu lançamento contábil reduz caixa e a conta de receber/pagar, sem tocar em receita, despesa ou quitação da compra original. O `payer_party_id` registra pagador externo quando confirmado; as obrigações alocadas registram quem devia ou a quem se devia. Se os papéis divergirem, não inferir que o pagador é o titular de todas as obrigações quitadas. Se o pagamento financeiro estiver ligado, valor, moeda, data efetiva e identidade de quem pagou precisam reconciliar com o evento de acerto.

## Integridade, índices e comportamento

- Organizador pode não consumir e pode não pagar. Igual divide somente pelos consumidores selecionados. Não criar automaticamente uma parcela minha.
- Participante, cota e contribuição são registros distintos: alguém pode consumir sem pagar ou pagar sem consumir. Cada Party aparece uma vez no grupo, `Self` é opcional e único, cada participante tem no máximo uma cota e as cotas somam o total do grupo.
- Se o grupo aponta para uma transação, exigir que ela esteja confirmada, seja uma despesa e pertença ao mesmo espaço; a transação continua sendo a fonte do valor, moeda e pagamentos ao fornecedor.
- Soma das shares igual ao total; contribuições registram o que já foi pago. Relações entre contatos servem ao rateio privado, sem vincular contas de usuários reais.
- Liquidação de obrigação não vira receita/despesa pessoal. Devolução comercial do fornecedor continua no fluxo Refund do pacote 07; ela é diferente de dinheiro devolvido por um contato.
- Cotas e contribuições são a fonte do plano de compensação; obrigações pessoais são materializadas de arestas únicas envolvendo eu. Arestas somente entre contatos são informativas e não entram no saldo pessoal.
- Cada obrigação de grupo corresponde a uma única aresta que envolve `Self`; a Party e a direção (`Receivable`/`Payable`) devem corresponder ao participante oposto. Obrigação direta não tem `group_id` nem `source_edge_id` e pode apontar para a transação de origem; obrigação originada de rateio usa grupo/aresta e não repete a transação nessa coluna.
- Registrar contribuição classifica um pagamento existente; criar obrigação não gera nova saída de caixa. Validar valor e vínculo da alocação para não duplicar integração. Grupo só organizado, sem fluxo próprio, não cria obrigação pessoal nem ledger.
- Saldo por contraparte é calculado pelas obrigações menos alocações de liquidação; consumo por contraparte vem das shares. Exibir esses dados separados e sem reclassificar a contraparte como tag.
- Uma liquidação pode alocar a várias obrigações na mesma direção e moeda. Se o pagador efetivo diferir das pessoas devedoras, registrar o tipo/identidade do pagador e as obrigações quitadas explicitamente. Não presumir que quem enviou o dinheiro é titular de todas as dívidas liquidadas.
- `SettlementPayment` representa liquidação efetiva; previsão de pagamento fica na obrigação, vencimento, tarefa ou notificação. Se houver `financial_payment_id`, data, valor, moeda, meio e pagador precisam reconciliar; o financeiro é fonte do método quando ligado.
- Se `financial_payment_id` estiver preenchido, tipo e identidade do pagador, data, valor, moeda e meio precisam corresponder ao `FinancialPayment`; `Contact` exige a mesma Party e `Self` exige a conta pessoal compatível com o sentido do fluxo. `Unknown` não pode produzir posting pessoal presumido.
- Se já houver liquidações, alterar o rateio exige ajuste versionado, sem apagar/recriar obrigações quitadas. Liquidação parcial bloqueia obrigações e revalida saldo; estorno preserva lançamento original.

## Conversão do James 1.x

TheyOwe/IOwe viram obrigações; TheyPaid/IPaid viram liquidações na direção correta. Preservar rateios e arquivamento. Reconciliação considera a antiga regra de saldo diário limitado a zero; diferença exige relatório.

## Componentes previstos

SaveSplit; CalculateShares; RecordContribution; SettleObligations; ReverseSettlement; ArchiveSettlement.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
