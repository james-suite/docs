---
id: acertos
title: Acertos e divisões sem parcela própria
description: Separação entre pagadores, participantes, cotas e liquidações de despesas.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, financas, cartoes, pessoas-empresas, compatibilidade-legado, migrations, migration-10-settlements, enum-payment-payer-type
source_refs: https://github.com/james-suite/james/blob/master/app/Services/SettlementGroupService.php, https://github.com/james-suite/james/blob/master/app/Services/SettlementBalanceCalculator.php, https://github.com/james-suite/james/blob/master/app/Http/Requests/StoreSettlementGroupRequest.php
updated: 2026-11-03
---

## Caso confirmado

O usuário pode pagar ou organizar uma despesa sem consumir uma parte. Exemplo: paga R$ 120 de três pessoas, cada uma deve R$ 40 e a parcela do usuário é zero. Também pode apenas organizar um rateio pago por contatos, sem qualquer movimento em suas contas.

Isso é divisão de despesas com contatos. Os contatos não precisam ter login nem correspondência com outros usuários do James. Colaboração entre contas reais de usuários fica fora da primeira versão.

## Modelo proposto

Separar o grupo de despesa, seus participantes, suas cotas de consumo, contribuições de quem pagou e liquidações posteriores. A pessoa que paga pode ser diferente de quem consome. O organizador não precisa constar na lista de consumidores. A identidade “eu” pertence ao espaço pessoal, sem criar um contato fictício obrigatório.

- Soma das cotas igual ao total; soma das contribuições registra o que já foi pago ao fornecedor.
- Divisão igual considera somente participantes selecionados para consumir. Não acrescentar “eu” automaticamente ao divisor.
- Divisão exata aceita parcela própria zero e verifica a soma em unidades monetárias mínimas.
- Um participante pode contribuir mais ou menos que sua cota; o saldo resulta de contribuição, consumo e liquidações efetivas.
- Múltiplos pagadores são previstos no schema; liberar a interface após validar os casos simples e o algoritmo de saldos.
- Pesos/percentuais são uma evolução do rateio, com regra explícita para sobra de centavos.

## Efeito no financeiro

A parte própria é consumo/despesa pessoal; o que foi pago por outras pessoas ou adiantado para elas compõe obrigações, não a parcela de consumo do usuário. Uma despesa apenas organizada não gera lançamento em conta pessoal. Cada integração é explícita e idempotente; editar uma observação não recria transações.

Acertos individuais continuam disponíveis: valor a receber, valor a pagar e liquidações parciais. Uma obrigação pode nascer de um rateio ou ser ligada diretamente à transação que originou o adiantamento/dívida. A contraparte é uma `Party` (pessoa ou organização), não uma tag.

### Pagamento de acerto não é uma tag nem uma nova despesa

Não criar uma tag genérica “reembolso”. Quando alguém devolve um adiantamento, registrar `SettlementPayment` alocado à obrigação daquela pessoa/empresa. Se o dinheiro entrar ou sair de uma conta pessoal acompanhada no James, ligar a liquidação a um `FinancialPayment` do tipo `Settlement`; esse movimento atualiza caixa e saldo a receber/pagar, sem alocação à compra original e sem gerar receita/despesa. Se o pagamento foi em dinheiro ou fora das contas acompanhadas, pode-se registrar a liquidação sem inventar movimentação bancária.

Cada liquidação identifica o pagador como usuário (`Self`), contato conhecido (`Contact`) ou desconhecido (`Unknown`). Para `Contact`, exigir a `Party` que enviou o dinheiro; para os outros casos, não associar uma contraparte presumida. Isso também deixa claro quem pagou em espécie fora de uma conta acompanhada, sem depender da direção da obrigação para adivinhar o pagador.

O contato/empresa relacionado à obrigação fornece a dimensão para filtros e gráficos. A tela pode agrupar por pessoa sem criar uma tag: consumo por participante vem das cotas; quem pagou ao fornecedor vem das contribuições; adiantamentos e pendências vêm das obrigações e alocações de liquidação; caixa recebido/pago vem dos pagamentos financeiros efetivados ou da liquidação registrada fora das contas. Exibir cada métrica separada e permitir abrir seus registros de origem.

Separar três situações: (1) uma pessoa devolve dinheiro de um adiantamento — liquidação de acerto, sem receita; (2) uma loja devolve valor de uma compra — devolução comercial ligada ao documento original, com `PaymentKind::Refund`; (3) uma empresa paga um adiantamento devido — liquidação de obrigação com uma organização como contraparte. Comprovantes anexados a uma liquidação ou rateio também viram Documents e podem ser consultados na biblioteca. Só registrar receita quando não houver uma obrigação anterior e a natureza do valor tiver sido confirmada como receita.

## Fonte única de saldos e integração

Cotas e contribuições geram um plano explícito de quem deve a quem. Somente as arestas que envolvem eu viram obrigações pessoais; relações entre contatos são informação do rateio, sem movimento nas minhas contas. O plano deve zerar os saldos líquidos de todos os participantes e manter uma revisão estável. Múltiplos pagadores só entram na UI quando esse plano estiver validado.

Uma contribuição paga pelo usuário aponta para a alocação do pagamento ao fornecedor. Não lançar caixa de novo ao registrar a contribuição ou materializar a obrigação. Uma liquidação posterior aponta para suas obrigações; quando houver movimento numa conta pessoal, o pagamento financeiro fica ligado a essa liquidação, nunca à compra original. Depois de liquidado, um rateio exige ajustes versionados em vez de excluir seus filhos e recriá-los.

Uma liquidação pode quitar parcialmente uma obrigação ou alocar um pagamento a várias obrigações na mesma direção e moeda. Se o pagador for diferente das pessoas devedoras, registrar quem enviou o dinheiro e quais obrigações foram quitadas de forma explícita antes de atualizar os saldos. O pagador efetivo e a contraparte da obrigação são dimensões diferentes para histórico e gráficos.

## Arquivamento

Arquivar um contato ou acerto oculta sua posição na triagem, preservando o saldo e o histórico. Antes de arquivar, mostrar pendências; o usuário decide arquivar mesmo assim. Exclusão não apaga liquidações nem obrigações já efetivadas.

## Compatibilidade

O 1.x já aceita `my_amount = 0` no rateio exato; a divisão igual possui regra específica que considera o usuário. Preservar os valores legados sem recalcular rateios passados segundo a nova regra. A parcela antiga pode ser reconstruída por total menos cotas dos contatos, confrontando os itens financeiros existentes. Não aplicar essa inferência se houver inconsistência.
