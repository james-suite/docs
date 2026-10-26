---
id: acertos
title: Acertos e divisões sem parcela própria
description: Separação entre pagadores, participantes, cotas e liquidações de despesas.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, financas, cartoes, pessoas-empresas, compatibilidade-legado, migrations
source_refs: https://github.com/james-suite/james/blob/master/app/Services/SettlementGroupService.php, https://github.com/james-suite/james/blob/master/app/Services/SettlementBalanceCalculator.php, https://github.com/james-suite/james/blob/master/app/Http/Requests/StoreSettlementGroupRequest.php
updated: 2026-10-25
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

A parte própria é despesa pessoal. Adiantamento para contatos gera valor a receber. Reembolso liquida esse crédito sem virar salário/receita pessoal. Uma despesa apenas organizada não gera lançamento no banco do usuário. Cada integração é explícita e idempotente; editar uma observação não recria transações.

Acertos individuais continuam disponíveis: dívida a receber, dívida a pagar e suas liquidações parciais. Guardar obrigações e pagamentos em linhas separadas permite explicar saldo sem limitar artificialmente um acumulador a zero. Saldo histórico legado será comparado separadamente quando a antiga regra produzir diferença.

## Fonte única de saldos e integração

Cotas e contribuições geram um plano explícito de quem deve a quem. Somente as arestas que envolvem eu viram obrigações pessoais; relações entre contatos são informação do rateio, sem movimento nas minhas contas. O plano deve zerar os saldos líquidos de todos os participantes e manter uma revisão estável. Múltiplos pagadores só entram na UI quando esse plano estiver validado.

Uma contribuição do usuário aponta para a alocação do pagamento ao fornecedor. Não lançar caixa de novo ao registrar a contribuição ou materializar a obrigação. Depois de liquidado, um rateio exige ajustes versionados em vez de excluir seus filhos e recriá-los.

## Arquivamento

Arquivar um contato ou acerto oculta sua posição na triagem, preservando o saldo e o histórico. Antes de arquivar, mostrar pendências; o usuário decide arquivar mesmo assim. Exclusão não apaga liquidações nem obrigações já efetivadas.

## Compatibilidade

O 1.x já aceita `my_amount = 0` no rateio exato; a divisão igual possui regra específica que considera o usuário. Preservar os valores legados sem recalcular rateios passados segundo a nova regra. A parcela antiga pode ser reconstruída por total menos cotas dos contatos, confrontando os itens financeiros existentes. Não aplicar essa inferência se houver inconsistência.
