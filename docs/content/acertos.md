---
id: acertos
title: Acertos
description: Controle de saldos entre contatos, lançamentos individuais e divisões de despesas com integração financeira opcional.
type: feature
status: observed
visibility: public
tags: acertos, dívidas, despesas, rateio
related: contatos, financas, auditoria, dashboard
source_refs: https://github.com/james-suite/james/blob/master/routes/settlements.php, https://github.com/james-suite/james/blob/master/app/Models/Settlement.php, https://github.com/james-suite/james/blob/master/app/Models/SettlementGroup.php, https://github.com/james-suite/james/blob/master/app/Models/ContactSettlementArchive.php, https://github.com/james-suite/james/blob/master/app/Enums/SettlementType.php, https://github.com/james-suite/james/blob/master/app/Services/SettlementBalanceCalculator.php, https://github.com/james-suite/james/blob/master/app/Services/SettlementGroupService.php
---

## O que o módulo resolve

Acertos registra obrigações informais entre o usuário e seus contatos: empréstimos, despesas pagas por alguém, reembolsos e pagamentos recebidos. Ele responde a duas perguntas diferentes:

- **Quem tem saldo a receber ou a pagar?** O módulo de competência calcula isso por contato.
- **Quando o dinheiro realmente entrou ou saiu de uma conta?** Essa parte pertence ao módulo de [Finanças](doc:financas) e só é criada quando o usuário escolhe integrar o lançamento.

Um acerto não é uma conta bancária nem uma fatura de cartão. Ele representa a relação entre pessoas; a transação financeira vinculada é opcional.

{{diagram:settlements-model}}

## Acerto individual

O fluxo começa em `/settlements`, segue para o contato e abre `/settlements/contact/{contact}/create`. Um lançamento individual possui:

| Campo | Regra |
| --- | --- |
| Contato | Obrigatório e escolhido no cadastro de [Contatos](doc:contatos). |
| Tipo | Um dos quatro tipos de movimento abaixo. |
| Valor | Obrigatório e maior que zero. Valores com vírgula são normalizados. |
| Descrição | Obrigatória, com até 255 caracteres. |
| Data | Data do fato ou do pagamento. |
| Anexos | Até 5 imagens ou PDFs, com até 10 MB por arquivo. |

### Tipos de movimento

| Tipo no código | Rótulo na interface | Efeito no saldo |
| --- | --- | --- |
| `they_owe` | Me deve | Aumenta o valor que o usuário tem a receber. |
| `they_paid` | Recebi pgto. | Reduz o valor que o usuário tem a receber. |
| `i_owe` | Eu devo | Aumenta o valor que o usuário tem a pagar. |
| `i_paid` | Realizei pgto. | Reduz o valor que o usuário tem a pagar. |

Os tipos de pagamento não apagam o lançamento original. Eles registram a quitação como um novo movimento, mantendo a sequência que explica como o saldo chegou ao valor atual.

## Como o saldo é calculado

O `SettlementBalanceCalculator` percorre os lançamentos em ordem de data e, dentro do mesmo dia, por ID. Ele acumula dois contadores em centavos:

- **A receber**: `they_owe` soma e `they_paid` subtrai.
- **A pagar**: `i_owe` soma e `i_paid` subtrai.

Depois de cada dia, cada contador é limitado a zero para impedir que um pagamento de um regime produza saldo negativo no outro. O resultado exibido para cada contato é:

```text
toReceive = saldo acumulado a receber
toPay     = saldo acumulado a pagar
netBalance = toReceive - toPay
```

Na prática:

- `netBalance > 0`: o contato deve ao usuário;
- `netBalance < 0`: o usuário deve ao contato;
- `netBalance = 0`: não há saldo pendente no recorte exibido.

O índice geral soma os saldos líquidos positivos e negativos de todos os contatos ativos. Contatos arquivados ficam fora da lista principal até o usuário escolher visualizar os arquivados.

## Quitar um saldo

Na tela do contato, o botão **Quitar dívida** prepara um novo lançamento com o saldo líquido atual:

- se o contato devia, o tipo sugerido é `they_paid`;
- se o usuário devia, o tipo sugerido é `i_paid`;
- o valor sugerido é o valor absoluto do saldo;
- a descrição padrão é `Quitação de saldo`.

A tela também oferece **Compartilhar**, que gera uma mensagem com o estado do saldo. Quando existem chaves PIX cadastradas nas contas financeiras, uma chave pode ser incluída na mensagem. O texto pode ser copiado ou aberto em uma conversa do WhatsApp; isso não envia a mensagem pelo James.

## Divisão de conta em grupo

Uma divisão de conta cria um `SettlementGroup` para a despesa e um `Settlement` filho para cada contato participante. O grupo guarda a descrição, o valor total, a data, o modo de rateio e, opcionalmente, a transação financeira vinculada.

O fluxo exige ao menos um contato, não permite o mesmo contato duas vezes e oferece dois modos:

- **Igual (`equal`)**: cada contato recebe a mesma parcela em centavos; a parte do usuário absorve eventual sobra do arredondamento.
- **Exato (`exact`)**: cada participante e o usuário informam suas próprias partes.

Em ambos os modos, a soma da parte do usuário com as partes dos contatos precisa ser exatamente igual ao total da despesa. Valores dos contatos precisam ser positivos; a parte do usuário pode ser zero.

Ao editar um grupo, o serviço atualiza os metadados e substitui os lançamentos filhos pelo novo rateio dentro de uma transação de banco. Um lançamento pertencente a grupo não pode ser editado ou excluído isoladamente; a edição deve ser feita no grupo.

## Integração com Finanças

O formulário oferece **Criar transação no módulo financeiro**. Quando habilitado:

1. o acerto individual cria ou atualiza uma `FinancialTransaction` ligada à conta ou à fatura do cartão escolhida;
2. `they_paid` vira uma receita financeira; os demais tipos usam uma despesa financeira;
3. um cartão resolve a fatura correspondente à data do lançamento;
4. tags podem ser escolhidas para uma transação criada a partir de `i_paid`, com uma tag principal entre as selecionadas;
5. uma divisão de conta cria uma despesa com um item `Minha Parte` e um item para cada contato, usando a tag protegida `Reembolso` nos itens dos participantes.

A relação é mantida por `financial_transaction_id`. O acerto continua sendo o registro da relação pessoal; a transação é a representação no caixa.

## Anexos

Acertos individuais e grupos aceitam anexos de imagem JPEG/PNG/JPG ou PDF. Cada operação aceita até cinco arquivos de no máximo 10 MB. Os arquivos ficam na coleção `attachments`, em disco privado, e aparecem no histórico com um indicador de anexos.

## Arquivamento, lixeira e histórico

Arquivar um contato em `/settlements/contact/{contact}` cria um registro em `contact_settlement_archives`. Isso não remove lançamentos: apenas tira o contato da visão principal dos acertos. A visão de arquivados e o histórico completo continuam disponíveis.

O módulo também possui:

- `/settlements/history`: histórico global paginado de lançamentos;
- `/settlements/groups`: lista de divisões de conta;
- `/settlements/trashed`: lixeira de acertos individuais;
- `/settlements/groups/trashed`: lixeira de grupos;
- restauração de grupos com seus filhos e sua transação financeira;
- soft delete nos lançamentos e grupos, com exclusão permanente nas operações específicas da lixeira.

Excluir um grupo trata seus lançamentos filhos como parte da mesma operação. A exclusão e a criação da transação vinculada são encapsuladas em transações de banco para evitar um grupo sem seus filhos ou uma integração financeira incompleta.

## Auditoria

`Settlement`, `SettlementGroup` e `ContactSettlementArchive` participam do activity log. Criação, alteração, exclusão e restauração são visíveis conforme o ciclo de vida de cada modelo. Consulte [Auditoria e logs](doc:auditoria) para investigar um lançamento ou uma divisão depois da operação.

### Referências

- [Contatos](doc:contatos) — cadastro dos participantes.
- [Finanças](doc:financas) — contas, cartões, transações e tags vinculadas.
- [Dashboard](doc:dashboard) — resumo dos saldos pendentes e notificações.
