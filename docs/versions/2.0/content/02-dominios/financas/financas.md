---
id: financas
title: Finanças — transações e múltiplos pagamentos
description: Fatos econômicos, itens extraídos de notas, vários pagadores, pagamentos e movimentações de saldo.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, cartoes, acertos, tags, ia-e-memoria, produtos-precos, migrations, enums, veiculos, migration-06-transactions, migration-07-payments, enum-payment-payer-type
source_refs: https://github.com/james-suite/james/blob/master/app/Models/FinancialTransaction.php, https://github.com/james-suite/james/blob/master/app/Models/FinancialAccount.php, https://brasilapi.com.br/docs
updated: 2026-11-07
---

## Proposta central

Manter a experiência de registrar uma transação com descrição, data, valor, itens, tags e anexos. Internamente, separar três conceitos: o que aconteceu (`financial_transactions`), como o valor se divide (`financial_transaction_items`) e como foi pago (`financial_payments` e alocações). Uma compra poderá receber vários pagamentos, de várias contas ou instrumentos, em datas diferentes.

Tags podem ser associadas ao cabeçalho da transação e, separadamente, aos itens. Transações simples sem itens continuam classificáveis no cabeçalho; em compras detalhadas, tags dos itens descrevem cada linha e as do cabeçalho descrevem a compra como um todo. A interface pode exibir o contexto combinado, mas não copia tags do cabeçalho para linhas nem duplica valores em relatórios.

Exemplo: uma compra de R$ 150 tem um pagamento de R$ 50 por Pix e outro de R$ 100 no cartão. Continua sendo uma compra de R$ 150, com duas alocações. Pagar depois a fatura de R$ 100 movimenta banco e dívida do cartão; não cria outra despesa de R$ 100.

## Organização da compra e dos instrumentos

Uma compra possui descrição, contraparte, data, valor e itens opcionais. A interface agrupa Itens, Pagamentos e Vínculos sob a mesma transação. Cada pagamento identifica valor, pagador, data e conta ou cartão. A tela de cartão consulta as compras financiadas por aquele cartão, suas parcelas e faturas.

Compra de R$ 300 com R$ 50 em dinheiro físico, R$ 70 por Pix e R$ 180 no crédito gera três pagamentos. Os R$ 180 podem virar três parcelas de R$ 60. O consumo total é R$ 300 na data econômica, o caixa sai R$ 50 da conta Dinheiro físico e R$ 70 da conta corrente, e a dívida do cartão cresce R$ 180. Cada parcela aparece na respectiva fatura e sua quitação reduz caixa e dívida.

O meio Pix/débito/boleto pode ser informado como detalhe opcional do pagamento. A proposta usa enum simples e dispensa cadastro de meios por conta. Crédito exige identificar o cartão. Dinheiro físico identifica a conta do tipo Wallet, cujo rótulo muda e mantém o valor persistido wallet.

## Invariantes

- Dinheiro usa unidades mínimas inteiras (`amount_minor`): R$ 10,25 vira `1025`. Não usar `float` em somas, rateios, juros ou conciliação. Quantidades e preços unitários podem ter precisão maior; o total é arredondado uma vez conforme a moeda.
- Cada operação financeira pertence ao espaço pessoal da conta autenticada. Contas, cartões, categorias, participantes e pagamentos relacionados devem pertencer ao mesmo espaço.
- O total concilia itens líquidos e o saldo de ajustes globais ainda sem atribuição. Arredondamento tem destino determinístico e visível, preservado em centavos.
- `occurred_on` representa o fato econômico; `due_on`, o vencimento; `posted_at`, a efetivação do movimento. Não reunir tudo em um campo de data.
- O ciclo do documento (rascunho, confirmado, cancelado) é separado de sua situação de pagamento, calculada pelas alocações válidas.
- Pagamentos efetivados são corrigidos por operação inversa e substituição rastreável. Não editar silenciosamente um pagamento antigo para acumular pagamentos novos.
- Pagamentos programados não alteram o saldo. A efetivação, a alocação, os lançamentos internos e a auditoria são confirmados juntos e com chave de idempotência.

## Leitura de nota fiscal e pagador

O usuário pode capturar uma NFC-e/XML ou imagem e revisar os campos extraídos antes de salvar. A leitura preenche contraparte, data e linhas com descrição, quantidade, preço, desconto e total; o arquivo original fica como Document e FiscalDocument guarda a extração identificada por versão. O registro começa como rascunho. Uma nota não deve efetivar sozinha um pagamento, escolher uma conta bancária ou inferir um veículo.

Cada pagamento declara quem pagou: usuário, contato ou desconhecido. Quando o usuário pagou, o instrumento pessoal é obrigatório e o efeito segue a conta ou o cartão selecionado. Quando um contato pagou, o custo pode ficar registrado sem criar débito pessoal; eventual obrigação nasce somente de uma decisão explícita no módulo de [acertos](doc:acertos). Custo do bem, consumo do usuário, contraparte e fluxo da conta são dimensões diferentes.

## Descontos gerais e atribuição parcial

Guardar o desconto original de cada linha e cada ajuste global da nota com procedência. Durante a revisão, permitir atribuir parte de um desconto global a um ou vários itens, informando valor e motivo. Mostrar o total do ajuste, quanto já foi atribuído e o saldo a distribuir.

Exemplo de compra com R$ 150 brutos e R$ 15 de desconto global. Carne de 2 kg custa R$ 80, dois cafés custam R$ 50 e detergente custa R$ 20. A pessoa atribui R$ 10 à carne. O líquido conhecido da carne passa a R$ 70 e R$ 35/kg. Os R$ 5 restantes continuam como desconto global a distribuir. O total da compra permanece R$ 135. A interface informa a pendência de atribuição no preço por produto.

A equação passa a ser total da transação = soma dos líquidos dos itens + acréscimos globais ainda sem atribuição − descontos globais ainda sem atribuição. Uma alocação transfere o efeito do ajuste para a linha. Total da compra e pagamentos permanecem os mesmos. Desconto original de linha já discriminado na nota entra uma única vez.

Uma nota pode ser confirmada financeiramente com ajuste global residual identificado e conciliado. O histórico de produto guarda o preço conhecido e a pendência de cobertura. O indicador Minha inflação considera somente observações com desconto/acréscimo atribuível resolvido. Permitir distribuir o saldo restante em outra revisão e atualizar derivados rastreáveis.

## Entrada simples sem documento fiscal

Uma compra ou despesa pode ser registrada sem nota, com descrição, data e valor no cabeçalho e itens opcionais. A proposta é aceitar uma frase natural e transformá-la em rascunho revisável; a pessoa confirma o lançamento pelo fluxo financeiro existente. Não criar nota fiscal ou itemização fictícia. Pagamentos continuam independentes: escolher “paguei” no texto não autoriza a IA a escolher conta/cartão nem a efetivar movimentação de saldo.

Se a pessoa informar produtos, relacionar as linhas a um produto de catálogo somente após correspondência confiável ou confirmação. O estabelecimento/filial faz parte do contexto de reconhecimento: um apelido ou código local de uma loja não deve identificar automaticamente produto de mesmo nome em outro lugar. Itens manuais confirmados podem alimentar o histórico de preços se produto, valor, quantidade/unidade e origem forem conhecidos.

Para abastecimento, a linha fiscal de combustível pode alimentar o histórico do [veículo](doc:veiculos) pelo seu vínculo de item. Se a nota tiver um único item total de combustível e desconto identificável, o valor líquido pode determinar o preço por litro. Um desconto geral numa nota com vários produtos exige alocação revisada, sem rateio automático presumido.

As linhas de mercado também podem ser relacionadas a um produto do catálogo pessoal para guardar código, marca, apresentação e unidade normalizada. A partir de compras confirmadas, [produtos e preços](doc:produtos-precos) mostram evolução do preço, comparação entre lugares e uma cesta pessoal “Minha inflação”. Isso consulta observações ligadas às linhas financeiras; não cria despesas duplicadas e não confunde o indicador pessoal com um índice oficial.

## Estratégia de leitura fiscal por estado

Manter uma fronteira de adaptadores por UF, com versão do leitor, jurisdição e campos reconhecidos registrados em cada tentativa. O scraper de RS existente deve ser inspecionado para reaproveitamento. Para outras UFs, começar por XML/dados estruturados e consultas fiscais suportadas, adicionando leitor determinístico quando houver documentos de exemplo para validar a cobertura.

Se não houver adaptador compatível ou ele falhar/incompletar os campos, oferecer extração por LLM como fallback explícito, usando a conexão de IA escolhida e a permissão específica para documentos fiscais. Enviar apenas o necessário, controlar custo/quota e identificar o processador. A extração do modelo é sempre candidata, exibe confiança por campo quando disponível e exige revisão humana. Sem consentimento, conexão ou saldo, permitir leitura manual; nunca trocar provedor silenciosamente, efetivar pagamento ou confirmar a transação.

## Associações com bens e serviços

Uma transação financeira pode ser associada a um veículo mesmo sem ter itens ou evento de serviço. A interface mostra o veículo como chip contextual e permite filtrar as finanças por veículo, para funcionar visualmente como uma tag; no modelo continua sendo uma relação tipada em `vehicle_financial_links`, não uma tag do catálogo. Para uma compra inteira destinada a um veículo, vincular a transação; para uma compra mista, vincular os itens correspondentes, inclusive a veículos diferentes. O vínculo não cria segundo documento ou pagamento nem transforma a compra em manutenção concluída. Ao trocar depois o óleo comprado, criar o evento com a data real e relacionar o item adquirido.

## Correção técnica e devolução comercial

Reverter um pagamento lançado errado pode reabrir saldo a pagar. Receber uma devolução de uma compra válida cria um documento relacionado e novo fluxo inverso, sem voltar a cobrar o fornecedor como se a compra estivesse impaga. Para pagamentos que cobrem vários documentos, o retorno identifica as alocações originais e respeita o limite cumulativo de cada uma.

Um documento confirmado só é cancelado sem pagamentos efetivos; se estava pago, a reversão técnica deve ser concluída antes. Compras pagas e depois devolvidas continuam no histórico com sua devolução vinculada.

## Integração financeira com acertos

A compra original continua sendo uma única `FinancialTransaction`. O rateio aponta para ela quando existe, distribui cotas de consumo e identifica quem pagou ao fornecedor. Minha parcela é consumo pessoal; o que adiantei por outras pessoas fica a receber, e o que outra pessoa pagou por mim fica a pagar. A relação com `Party` permite filtrar e gerar gráficos por contato ou empresa sem atribuir tag de pessoa.

Quando alguém paga uma obrigação, criar uma `SettlementPayment` e alocá-la à obrigação. Se o valor passar por uma conta pessoal acompanhada, ligar um `FinancialPayment` do tipo `Settlement`; esse pagamento lança caixa contra receber/pagar. Ele não recebe alocação à transação original e não altera a situação de quitação da compra. Liquidação registrada em dinheiro ou fora das contas acompanhadas pode ficar sem `FinancialPayment` e sem posting pessoal.

Gráficos de acertos devem oferecer medidas separadas: consumo por participante, valores adiantados, saldo aberto por contraparte, pagamentos recebidos e pagos. O filtro por pessoa pode cruzar essas medidas com transações, produtos e veículos relacionados, mas o relatório deve deixar claro se mostra consumo, dívida ou caixa. Não somar os três como “gasto por pessoa”.

## Controle de saldo proposto

Adotar um livro interno simplificado de lançamentos balanceados, com contas de ativo, passivo e contrapartidas técnicas. O usuário continua vendo contas, cartões e transações; não precisa operar um plano contábil na interface. Cada lote efetivado deve ter débitos e créditos iguais por moeda. Esse desenho é uma proposta técnica a validar com um protótipo de pagamentos e importação antes de fechar o schema.

Compra no cartão aumenta seu passivo; transferência entre bancos altera os dois saldos sem gerar receita/despesa; pagamento de fatura reduz o passivo e o saldo bancário. Juros e tarifas são fatos próprios. O diário financeiro interno é diferente do diário pessoal.

## Relatórios

Oferecer visões com nomes claros: consumo/despesas pessoais, fluxo de caixa, compromissos futuros, dívidas de cartão e valores de terceiros. Liquidação de acerto não é receita ou despesa; devolução comercial tem vínculo à compra original; pagamento de fatura não é uma segunda despesa. Cada indicador deve declarar sua base, contraparte e período.

Transações financeiras de terceiros podem ser acompanhadas nos acertos sem inflar o consumo pessoal. Um gasto de R$ 100 pago pelo usuário, com R$ 100 devidos por contatos, representa saída de caixa e crédito a receber; a despesa pessoal é zero.

## Consumo, devoluções e base dos gráficos

A transação e os itens identificam consumo próprio, de terceiros, misto ou desconhecido. O pagador é uma dimensão independente. Em rateio confirmado, cotas são a fonte dos valores de consumo. Até existir atribuição por item, o relatório detalhado apresenta a parte de consumo ainda sem categoria/item conhecida. A consulta pode oferecer distribuição proporcional explicitamente identificada como estimativa.

Uma devolução comercial preserva a compra e cria transação relacionada de propósito Refund, direção econômica oposta e valor positivo. O relatório de consumo apresenta o valor como redução da natureza original na data da devolução. A visão de receita ordinária separa essas devoluções. Reembolso parcial por produto mantém a referência aos itens envolvidos, com cobertura pendente quando o fornecedor informar só o total.

FinancialPayment de tipo Refund se aloca ao documento de devolução e referencia a alocação original da compra para controlar limites. Crédito no cartão reduz a dívida daquele cartão. Valores ainda sem devolução efetiva conservam a distinção entre redução econômica, recebimento pendente e caixa efetivo.

## Arquivamento e histórico de instrumentos

Antes de arquivar conta ou cartão, mostrar saldo, parcelas futuras, faturas abertas, pagamentos agendados e recorrências. A pessoa escolhe tratar as pendências ou preservar o instrumento arquivado com pendências visíveis. Arquivamento impede novas compras usuais e mantém operações de liquidação, reversão e conferência de fatos anteriores. Reativação restaura a seleção para novas operações.

Permitir registrar um fato histórico anterior ao arquivamento por uma ação específica com aviso e autorização, mantendo trilha da operação. Seletores padrão mostram ativos, e filtros de histórico oferecem ativos e arquivados. O valor patrimonial e as obrigações continuam calculados inclusive para instrumentos arquivados.


Conta bancária conjunta entre usuários não faz parte do escopo inicial. O caso confirmado de “conta compartilhada” é [divisão de despesas sem parcela própria](doc:acertos).

Ao cadastrar uma conta, a instituição brasileira pode ser pesquisada pelo catálogo de bancos da BrasilAPI e vinculada ao cadastro de instituição correspondente. É apenas preenchimento de referência; não autoriza conexão bancária, leitura de saldo ou movimentação externa. Se o catálogo não localizar a instituição, manter o nome digitável.

## Compatibilidade 1.x

Preservar descrição, datas, valores, itens, recorrências, anexos e IDs de origem. O status legado `draft` vira rascunho; `pending` vira documento confirmado com pagamento pendente; `posted` exige analisar se representa movimento de banco ou compra de cartão. Transferências pareadas devem virar uma operação única com duas pernas, sem duplicar valor.

Não reconstruir uma compra parcelada inteira agrupando textos parecidos. Preservar cada parcela legada e sua data. Migrar separadamente tags do cabeçalho e dos itens, sem inferir tags de item a partir da transação. Permitir agrupamento posterior confirmado pelo usuário. O relatório de importação deve comparar os saldos e os relatórios antigos com os novos antes de liberar automações.

## Casos de aceite

- Compra com Pix e cartão, pagamento parcial em dias diferentes e um pagamento que quita várias compras.
- Transferência, estorno parcial, juros, desconto e cancelamento sem duplicar caixa ou despesa.
- Arquivar uma conta com saldo e um cartão com dívida sem perder histórico nem pendências.
- Repetir uma requisição ou job sem duplicar pagamentos ou lançamentos.
- Importar sem alteração silenciosa de saldo e sem disparar rotinas externas.
