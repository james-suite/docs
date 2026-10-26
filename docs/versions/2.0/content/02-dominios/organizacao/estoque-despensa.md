---
id: estoque-despensa
title: Estoque e despensa
description: Cadastro amplo de itens, localização física, quantidades, lotes, validade e reposição.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio, estoque
related: planejamento-2-0, financas, produtos-precos, veiculos, patrimonio, notificacoes, migrations, enums
source_refs:
updated: 2026-10-26
---

## Escopo do módulo

Tratar a despensa como estoque pessoal de qualquer coisa que alguém queira localizar ou controlar: alimentos, bebidas, limpeza, higiene, papelaria, materiais de trabalho/artesanato, peças, ferramentas e equipamentos guardados. Categorias são pessoais e hierárquicas; não limitar o sistema a supermercado ou cozinha.

O módulo responde “o que tenho, quanto tenho, onde está, em que condição e o que precisa de reposição?”. Cada usuário enxerga somente o estoque do próprio espaço. A existência de um item no estoque não transforma automaticamente o item em patrimônio, gasto, dívida ou bem de veículo.

## Produto comercial e item controlado

Separar o produto do catálogo comercial do item que a pessoa controla no próprio estoque:

- Product descreve algo comprável, sua marca, apresentação e códigos, podendo ter histórico de preços.
- StockItem representa a unidade de controle escolhida pela pessoa, sua categoria, unidade, política de lotes, quantidade mínima e local padrão.
- O vínculo com Product é opcional. Pode haver StockItem sem catálogo, compra ou GTIN; uma descrição da nota também não cria estoque sem confirmação.
- A mesma identidade comercial pode exigir mais de um StockItem pessoal quando cada uso ou regra de reposição for diferente. Variantes de embalagem continuam distintas no catálogo, mas podem ser convertidas para uma unidade-base comum.

## Lugares e mapa físico

Permitir uma árvore arbitrária de locais: residência → cozinha → armário → prateleira; garagem → estante → caixa; ou veículo → porta-malas. O nome é editável, e o tipo opcional facilita filtros sem impor profundidade fixa. Um item ou lote pode estar em vários locais ao mesmo tempo, porque cada saldo vem das linhas de movimentação por local.

Uma localização pode guardar ambiente descritivo (ambiente, refrigerado, congelado ou controlado), observações e posição na ordem da árvore. Isso ajuda a organizar e sugerir conservação, mas não representa leitura de sensor. Local “não informado” é um resultado visível de busca, não um local fictício.

Pesquisar por nome, código, categoria, local, lote, validade, condição ou fornecedor deve responder com saldo atual e caminho completo do local. Mover algo é uma operação explícita com origem e destino; arquivar um local com saldo exige primeiro mover ou contar seus itens. Não apagar locais usados no histórico.

## Unidades e formas de controle

Usar unidades-base exatas e conversões racionais do catálogo MeasurementUnit. Suportar contagem, massa, volume e comprimento conforme os itens; não misturar famílias nem arredondar o saldo armazenado. A UI pode exibir “2 pacotes de 500 g” enquanto o saldo interno usa 1 kg. Quantidade fracionária só é permitida para unidade/família que comporta fração.

Cada StockItem escolhe uma modalidade:

- **Agregado:** saldo total por local para itens simples, como detergente, arroz sem lote acompanhado ou parafusos.
- **Lote:** separa recebimentos e datas, útil para alimentos, cosméticos, materiais com validade, números de série de fabricação ou códigos de lote.
- **Individual:** acompanha cada unidade física por etiqueta ou número de série, condição, garantia e local, útil para ferramenta, aparelho e equipamento. Planejado na etapa avançada.

As quantidades são sempre expressas na unidade-base do item. Pacotes parcialmente abertos continuam contabilizados; registrar abertura e, quando conhecido, prazo de uso após abertura. Alterar unidade-base ou modalidade exige conversão/recontagem revisável e nunca reinterpreta movimentos passados.

## Movimentações e saldo

Movimentos são a fonte de verdade: entrada, consumo/uso, transferência, ajuste de inventário, perda/descarte, empréstimo e devolução. O saldo por item, lote e local é a soma de linhas confirmadas; não se edita um campo de saldo à mão. Uma transferência grava saída e entrada de mesmo item/lote e quantidade em um único evento, conservando o total.

Entradas manuais, estoque inicial, presente, importação ou compra fora do módulo financeiro são válidas. Ajustes de contagem, perdas, descarte e correções exigem motivo. Corrigir evento confirmado cria movimento inverso/compensatório e uma nova operação; não apagar a linha original. Bloquear saldo negativo por padrão e permitir divergência somente em estado de revisão até a pessoa corrigi-la.

Separar saldo físico, reservado e disponível: disponível = saldo físico − quantidade reservada. Reserva não baixa o saldo físico; seu consumo confirmado gera um movimento de saída. Empréstimo de unidade reutilizável registra quem recebeu, quando saiu e quando voltou.

## Vínculo opcional com finanças e preços

Ao revisar uma transação ou nota fiscal, oferecer a ação explícita “Adicionar ao estoque” para cada linha aplicável. A pessoa escolhe StockItem existente ou cria um, confirma quantidade, unidade, lote/validade e local. Uma linha financeira pode ser dividida entre várias entradas e locais; a quantidade total adicionada nunca supera a quantidade comprada convertida.

O fluxo inverso também existe: registrar uma entrada no estoque sem criar ou alterar transação financeira. Isso cobre estoque já existente no começo do uso, presente, consumo próprio, compra não lançada no James, amostra ou correção de inventário. O vínculo financeiro é opcional e nunca é requisito para buscar ou movimentar o estoque.

Movimento de estoque não lança despesa, pagamento, dívida nem receita. Se ele aponta para um FinancialTransactionItem, aquele item continua sendo o único fato financeiro. Histórico de preços continua vindo da linha financeira e seu preço revisado; receber o mesmo produto no estoque não cria outra observação de preço. A associação financeira é auditável e pode ser revista sem reescrever a transação.

## Lotes, validade e descarte

Para itens em modalidade Lote, guardar código do lote quando disponível, fabricação, validade, melhor consumo antes de, recebimento, abertura e observações. “Validade” e “consumir de preferência antes de” são campos diferentes. Datas desconhecidas permanecem vazias; o sistema não as inventa.

Na baixa, sugerir FEFO (primeiro a vencer, primeiro a sair) entre lotes disponíveis, sem impedir escolha manual. Alertar para itens próximos do limite configurado e exibir vencidos como pendência de decisão; nunca consumir, apagar ou descartar automaticamente. Descarte é movimento próprio com quantidade, local/lote, data e motivo.

Quando um prazo curto após abertura fizer sentido, guardar esse intervalo no StockItem ou no produto e calcular uma data sugerida a partir da abertura confirmada. A pessoa pode corrigir a data; alertas mostram a origem e não confundem validade impressa com prazo após abertura.

## Inventário, reposição e notificações

Contagens podem abranger local, categoria, itens selecionados, lotes ou unidades. Uma sessão guarda o saldo esperado no início e o saldo contado, permite contagem às cegas e revisão das diferenças. Movimentos feitos durante a contagem continuam no histórico; ao confirmar, recalcular a divergência com esses movimentos e gerar ajustes explícitos, sem sobrescrevê-los.

StockItem pode definir ponto mínimo, quantidade-alvo, local padrão e fornecedor preferencial. O painel sugere reposição pela quantidade disponível; a pessoa ajusta e envia para uma lista. Listas de reposição aceitam itens livres ainda sem cadastro. Marcar algo como comprado não altera estoque: a entrada ocorre quando a pessoa registra o recebimento ou associa a linha financeira.

Alertas de baixo estoque, validade, contagem pendente e empréstimo vencido usam o sistema de notificações existente, com preferências e deduplicação. O painel explica a causa e oferece uma ação útil; consultar uma página não cria notificações repetidas.

## Itens duráveis e ligações com outros módulos

Itens individuais podem guardar número de série, etiqueta interna, marca/modelo, condição, garantia, manual e anexos privados. Um item de alto valor pode se relacionar opcionalmente com Asset; uma peça ou acessório pode apontar para Vehicle ou evento de veículo. Esses vínculos não duplicam patrimônio, preço de compra, serviço nem despesa. A localização no estoque permanece independente da categoria contábil.

Kits e receitas podem futuramente reservar ingredientes e consumir quantidades somente após confirmação da preparação. Para o primeiro desenho, listas, reservas e relações são opcionais e não devem tornar a entrada simples de uma compra mais lenta.

## Interface e etapas

O painel de estoque começa por busca e respostas rápidas: saldo por local, sem local, abaixo do mínimo, lotes próximos da validade, movimentações recentes e contagens abertas. Ações rápidas: receber, consumir/usar, mover, contar e descartar. Formulários mobile-first aceitam código de barras/manual, unidade compreensível, quantidade, local e data; campos avançados aparecem só quando modalidade e contexto pedirem.

1. **Base utilizável:** categorias e árvore de locais, cadastro livre e ligado a Product, saldo agregado, movimentos manuais, transferência, contagem e vínculo financeiro opcional.
2. **Alimentos e consumíveis com rastreio:** lotes, unidades fracionadas, abertura, validade, FEFO, alertas e listas de reposição.
3. **Estoque avançado:** unidades individuais/serializadas, empréstimos, reservas para tarefas ou eventos, etiquetas QR próprias e integração opcional com kits/receitas.

IA pode sugerir o produto, categoria, unidade, lote ou quantidade lidos de uma nota, imagem ou código; cada dado precisa de confirmação. IA não dá entrada, altera saldo, decide descarte ou compra automaticamente.
