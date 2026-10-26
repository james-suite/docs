---
id: veiculos
title: Veículos, abastecimento e histórico
description: Cadastro opcional como bem, abastecimentos por nota fiscal, serviços, despesas e diário do veículo.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, financas, pessoas-empresas, agenda-tarefas, patrimonio, migration-15-vehicles, migration-18-assets, enum-vehicle-event-type
source_refs:
updated: 2026-10-26
---

## Veículo e patrimônio são registros independentes

O cadastro de veículo existe por si só para guardar abastecimentos, quilometragem, serviços, documentos e notas. Criar um veículo não obriga a incluí-lo no módulo de patrimônio. Quando fizer sentido acompanhar seu valor patrimonial, um bem pode apontar para aquele veículo; essa relação é opcional e única. Sem o vínculo, o veículo continua completo para uso e histórico.

Se for vinculado como bem, a avaliação entra uma vez no patrimônio. O mesmo veículo não pode aparecer de novo como um segundo bem sem vínculo. Compra, manutenção e venda continuam sendo fatos financeiros próprios, não avaliações patrimoniais.

## Nota fiscal e abastecimento

O fluxo deve aceitar uma NFC-e/XML ou imagem da nota. A leitura extrai candidatos para emitente, data, itens, quantidade, preço, descontos e total, preservando o arquivo original e a procedência. O resultado começa como rascunho para conferência: reconhecer a nota não efetiva pagamento nem adivinha o veículo quando houver dúvida. A chave fiscal ou uma impressão digital evita cadastrar o mesmo documento duas vezes.

Quando a nota contém um único item de combustível com quantidade identificável, o desconto desse item reduz o preço efetivo por litro: total líquido dividido pelos litros, com precisão decimal e arredondamento monetário definido. Se houver vários itens, o sistema não distribui automaticamente um desconto geral entre combustível e outros produtos; pede uma alocação explícita ou mantém o desconto sem atribuição ao combustível. Os valores brutos, desconto, líquido e quantidade permanecem consultáveis.

Após selecionar ou confirmar o veículo, a leitura pode criar juntos o rascunho financeiro com itens fiscais e o evento de abastecimento, ligados à mesma linha de item. A data da nota é a data da compra; a data em que a imagem foi lida fica como metadado de captura. Dados incertos devem ser revisados antes da confirmação.

## Quem pagou

A nota fiscal comprova a compra, mas não necessariamente quem entregou o dinheiro nem de qual conta saiu. A confirmação registra se pagou o usuário, um contato ou se o pagador ainda é desconhecido. Só o pagamento feito com uma conta pessoal movimenta o saldo dessa conta. Pagamento por contato não cria débito bancário pessoal e também não cria automaticamente uma dívida: eventual acerto/reembolso exige vínculo explícito no módulo de [finanças e acertos](doc:acertos).

Uma transação pode ter vários pagamentos e pagadores, cada um com sua própria data e instrumento. Isso permite guardar o custo real do veículo mesmo quando o usuário não pagou, sem misturar custo de uso, parcela pessoal e fluxo de caixa.

## Despesas, compras e serviços

Finanças continua sendo a fonte do valor e dos pagamentos. O vínculo de uma transação ou de um item com o veículo permite consultá-lo nos dois módulos sem criar uma segunda despesa. Uma compra de óleo pode ser marcada como “para este veículo” no dia da compra sem registrar troca de óleo naquele dia. Quando a troca acontecer, criar então o evento de serviço com sua data e quilometragem reais e relacionar o item comprado, se for possível.

Uma visita à oficina pode ter várias operações realizadas, peças, mão de obra, prestador e notas. Um evento pode se ligar a itens financeiros comprados antes ou a uma transação da própria oficina. A data do custo e a data do serviço permanecem distintas; o link financeiro não afirma que o trabalho foi executado. Para isso existe o evento efetivamente registrado. Tarefas e alertas também não comprovam serviço.

## Abastecimentos e odômetro

Guardar tipo de combustível, quantidade decimal, tanque cheio/parcial, posto, veículo e leitura do odômetro. O valor líquido vem da linha financeira vinculada quando disponível; sem comprovante, o evento pode ficar sem valor. Cálculo de consumo declara o intervalo e usa abastecimentos parciais entre dois tanques cheios; leitura ou volume ausente não produz média confiável.

Leituras são observações datadas com unidade e origem. Regressão do odômetro exige correção justificada ou troca do instrumento; não bloquear permanentemente um histórico real nem corrigir silenciosamente leituras passadas.

## Serviços e diário genérico

O histórico do veículo serve também como um diário simples de acontecimentos: instalar acessório, notar barulho, emprestar o carro, trocar pneu ou registrar qualquer outra coisa relevante. Um evento do tipo `Note` aceita título, texto livre, data e quilometragem opcionais. Ele não é automaticamente uma despesa, ordem de serviço ou manutenção concluída.

Serviços e reparos usam eventos próprios com vários itens realizados e observações. Anexos, prestador, nota fiscal e leitura de odômetro são opcionais. A pessoa pode registrar uma ação sem custo conhecido, ou um custo sem declarar que houve serviço naquele dia.

## Checklist e manutenção futura

Usar tarefas/checklists para itens pendentes: “verificar freios”, “trocar pneu”, “documentação”. Plano de manutenção guarda intervalo de tempo, distância ou ambos e a leitura/data de referência. Quando ambos existirem, mostrar o que vencer primeiro; concluir a tarefa não comprova serviço feito sem vincular um evento de manutenção real.

Modelos de checklist geram cópias para a execução. Alterar o modelo não muda o que foi conferido no passado. Itens podem ser concluídos, não aplicáveis ou reabertos com histórico. Lembrete cria pendência, não registra automaticamente uma manutenção.

## Etapas

Primeiro: cadastro independente de patrimônio, leitura de nota com revisão, vínculo financeiro, abastecimento, odômetro, eventos com vários itens e notas genéricas. Depois: recorrências por distância, alertas consolidados, pneus/componentes e análise de custo por km. Telemetria, localização automática e conexão com fabricante exigem integrações próprias.
