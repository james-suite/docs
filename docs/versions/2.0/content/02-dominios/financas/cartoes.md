---
id: cartoes
title: Cartões, faturas e parcelamentos
description: Ciclos históricos, limite, vários pagamentos de fatura e estornos.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, financas, migracao-2-0, compatibilidade-legado, migrations, enums
source_refs: https://github.com/james-suite/james/blob/master/app/Models/FinancialCreditCardInvoice.php, https://github.com/james-suite/james/blob/master/app/Models/FinancialCreditCard.php
updated: 2026-10-25
---

## Melhorias propostas

Cartão tem instituição emissora opcional, nome, últimos dígitos opcionais, limite, moeda e regras de fechamento/vencimento. Não guardar PAN completo, CVV ou credenciais bancárias. A conta padrão de pagamento é uma sugestão; uma fatura poderá ser paga por várias contas e em várias datas.

A fatura congela suas datas efetivas e sua competência de referência. Alterar as regras do cartão só afeta ciclos futuros ainda não consolidados. Manter a convenção atual de compra no próprio dia do fechamento pertencer à fatura que fecha naquele dia, com possibilidade explícita de ajustar o ciclo real.

## Estados independentes

O estado do ciclo é aberto, fechado ou cancelado. A situação de quitação (sem pagamento, parcial, quitada, crédito) e o atraso são derivados de valores e datas. Assim, uma fatura pode estar fechada, parcialmente paga e atrasada simultaneamente sem um enum contraditório.

Fechamento guarda o total consolidado e a revisão utilizada. Ajustes posteriores exigem lançamento identificado ou reabertura explícita; não mudar silenciosamente um demonstrativo já conferido.

## Pagamento de faturas

Cada pagamento é uma operação própria com data, conta de origem, valor e alocação à fatura. Um Pix de R$ 300 e outro de R$ 200 preservam as duas datas. Quitar não deve reescrever a conta e o status de todas as compras para simular movimento bancário.

Pagamento excedente precisa de destino explícito: crédito do cartão ou rejeição, conforme suporte da primeira versão. Juros, multa e tarifa são transações separadas e vinculadas à fatura; não confundir principal pago com encargo. Estornar pagamento cria operação inversa auditada.

## Parcelamentos e limite

Modelar plano de parcelas, número, valor, datas previstas e vínculo de cada parcela à fatura. Valor total e soma das parcelas devem coincidir; o arredondamento usa regra determinística. Distinguir compra parcelada de parcelamento/financiamento da fatura, que cria obrigação e encargos próprios.

O plano deve definir o limite comprometido considerando parcelas futuras e créditos, e deixar esse critério visível. Uma compra parcelada não pode ser contada novamente toda vez que uma parcela entra na fatura. Estornos parciais mantêm referência à compra e à parcela; reembolso posterior ao fechamento pode aparecer em outro ciclo sem apagar o histórico.

## Entrega em etapas

Primeiro: ciclos corretos, várias contas/pagamentos, arquivamento, parcelas e estornos básicos. Depois: conferência/importação de demonstrativos, cartões adicionais, limites compartilhados entre instrumentos e financiamento de fatura. Não prometer sincronização bancária ou cálculo de juros contratuais sem requisitos próprios.

## Migração e validação

O James atual possui `amount_paid`, `paid_at`, `payment_transaction_id` e `interest_transaction_id`. Um pagamento parcial agregado não contém necessariamente as datas individuais originais. Importar o agregado como pagamento legado, declarar a precisão disponível e não inventar histórico.

Quando a fatura foi integralmente paga, o 1.x pode ter efetivado suas compras na conta pagadora. A conversão precisa remover a dupla representação do caixa ao criar os pagamentos de fatura; reconciliar o total debitado antigo com o novo livro antes de confirmar. Inconsistências vão para relatório e não são compensadas silenciosamente com um ajuste de saldo.
