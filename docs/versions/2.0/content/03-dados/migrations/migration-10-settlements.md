---
id: migration-10-settlements
title: Migration 10 — Rateios e liquidações de acertos
description: Contrato lógico de tabelas, FKs, índices e conversão: rateios e liquidações de acertos.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, acertos, componentes-tecnicos
source_refs:
updated: 2026-10-25
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 03, 06, 07. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `settlement_groups`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, description, occurred_on, total_minor, currency, mode` | Despesa e modo de rateio; organizador implícito é o dono do espaço. |

## `split_participants`

| Campo ou grupo | Contrato |
| --- | --- |
| `group_id, party_id, is_self` | Parte do mesmo espaço ou identidade eu; XOR entre party_id e is_self. |
| `display_snapshot` | Nome no momento da operação, sem perder referência atual. |

## `split_shares`

| Campo ou grupo | Contrato |
| --- | --- |
| `participant_id, amount_minor, weight` | Consumo. Minha cota pode ser zero; consumidor terceiro tem valor positivo quando incluído. |

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
| `record_id, party_id, direction, amount_minor, currency` | Obrigação individual de receber/pagar. |
| `group_id, source_edge_id, occurred_on, due_on, archived_at` | Origem opcional; source_edge_id único materializa só arestas que envolvem o usuário, sem duplicar obrigação. |

## `settlement_payments / settlement_payment_allocations`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, paid_on, amount_minor, financial_payment_id / obligation_id, amount_minor` | Liquidação própria com alocação a uma ou várias obrigações. Integração bancária opcional e explícita. |

## Integridade, índices e comportamento

- Organizador pode não consumir e pode não pagar. Igual divide somente pelos consumidores selecionados. Não criar automaticamente uma parcela minha.
- Soma das shares igual ao total; contribuições registram o que já foi pago. Relações entre contatos servem ao rateio privado, sem vincular contas de usuários reais.
- Crédito a receber e reembolso não viram despesa/receita pessoal. Cotas e operações financeiras são ligadas sem efeitos duplicados.
- Cotas e contribuições são a fonte do plano de compensação; obrigações pessoais são materializadas de arestas únicas envolvendo eu. Arestas somente entre contatos são informativas e não entram no saldo pessoal.
- Registrar contribuição classifica um pagamento existente; criar obrigação não gera nova saída de caixa. Validar valor e vínculo da alocação para não duplicar integração. Grupo só organizado, sem fluxo próprio, não cria obrigação pessoal nem ledger.
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
