---
id: patrimonio
title: Patrimônio e bens
description: Bens, avaliações e documentos sem duplicar valores financeiros ou de veículos.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, veiculos, financas, migrations
source_refs:
updated: 2026-11-14
---

## Contrato

Guardar bem, categoria configurável, estado de posse, datas de aquisição/alienação, preço de aquisição e avaliações datadas com fonte. Associar empresa/pessoa da compra, Documents e transação existente; anexos enviados pelo patrimônio também aparecem na biblioteca documental. O veículo pode existir e ter histórico completo sem ser cadastrado como patrimônio; se o usuário quiser acompanhar seu valor, pode vinculá-lo a um único bem, sem duplicar cadastro nem custo.

Avaliação patrimonial é uma observação; valorização ou depreciação não representa entrada/saída de dinheiro. O total patrimonial precisa declarar se usa custo ou avaliação e sua data. Contas de investimento já incluídas no financeiro não devem aparecer de novo como bem equivalente.

Manutenção, garantia, número de série e localização atual são extensões úteis, opcionais. Histórico de propriedade permanece após arquivar o bem. Alienação pode vincular uma receita real, sem fabricar uma automaticamente.


## Investimentos por quantidade e custo — proposta

O usuário quer acompanhar investimentos com fontes gratuitas. A experiência inicial permite cadastrar o ativo, a instituição onde está custodiado, quantidade, data e valor total pago. O preço unitário pode ser informado e conferido contra o total. Taxas da operação ficam discriminadas. Para MXRF11, a posição acompanha as cotas detidas e as cotações disponíveis. A conta de investimento representa o caixa da instituição; seus ativos têm posições próprias.

### Posição e movimentações

Registrar compras, vendas, rendimentos recebidos, amortizações, transferências de custódia e eventos que alteram quantidade/custo. Quantidade e custo remanescentes derivam das movimentações, com precisão decimal explícita e aritmética exata. Custo médio gerencial considera os custos de aquisição definidos na regra da classe de ativo. Venda reduz quantidade e baixa o custo correspondente, permitindo apurar resultado realizado. Ajustes e eventos corporativos preservam origem e histórico.

A tela pode mostrar quantidade, custo remanescente, custo médio, última cotação com data/fonte, valor de mercado, resultado ainda não realizado, resultado realizado e rendimentos recebidos. A fórmula de avaliação é quantidade multiplicada pela cotação compatível com a unidade do ativo. Apresentar cobertura e data dos preços usados no total. Cotação ausente conserva a lacuna identificada; uma fonte indisponível conserva a última cotação com indicação de desatualização.

Exemplo sem taxas ou vendas, com números ilustrativos: compra de 100 cotas a R$ 10 resulta em custo de R$ 1.000. Uma cotação de R$ 9,80 avalia a posição em R$ 980 e mostra resultado não realizado de -R$ 20. Um rendimento efetivamente recebido de R$ 10 aparece separado, compondo resultado econômico acumulado de -R$ 10 nesse exemplo. Percentuais de rentabilidade exigem metodologia própria que considere datas e aportes/resgates.

Rendimentos declarados e recebidos têm estados próprios. Um rendimento recebido pode aumentar o caixa da corretora e vincular-se à entrada financeira existente. Reinvestimento registra o recebimento e uma compra distinta. Amortização recebe regra específica para custo e quantidade conforme o evento, com revisão dos dados de origem.

### Fontes gratuitas e atualização

Avaliar fontes gratuitas por cobertura do ativo, atraso, formato e condições de uso. A cotação diária atende ao acompanhamento patrimonial proposto. O scheduler consulta em lote apenas os ativos necessários, guarda a observação datada e reutiliza os preços entre telas. Provedores são substituíveis, com registro de origem, horário de referência, moeda e natureza do preço. Toda posição permite cotação manual datada e importação de arquivo quando faltar integração gratuita adequada.

| Fonte candidata | Aplicação e condição de avaliação |
| --- | --- |
| [Cotações históricas da B3](https://www.b3.com.br/pt_br/market-data-e-indices/servicos-de-dados/market-data/historico/mercado-a-vista/cotacoes-historicas/) | Arquivos COTAHIST em ZIP/TXT, com layout publicado e preços sem ajuste por proventos. Avaliar importação de arquivos e canal permitido para atualização. O [hub público](https://www.b3.com.br/pt_br/dados/hub-de-dados-publicos/) declara restrições de coleta automatizada. |
| [Banco Central, série Selic](https://dadosabertos.bcb.gov.br/dataset/11-taxa-de-juros---selic) | Série oficial para referência e cálculos compatíveis com o produto contratado. Preservar unidade e periodicidade da taxa, intervalo e metodologia do cálculo. |
| [Tesouro Transparente](https://www.tesourotransparente.gov.br/ckan/dataset/taxas-dos-titulos-ofertados-pelo-tesouro-direto) | Preços e taxas por título, vencimento e data. Usar a natureza de preço apropriada à avaliação desejada e informar o critério na tela. |
| [brapi](https://brapi.dev/docs) | Candidata se a cobertura e a quota gratuitas atenderem aos ativos escolhidos. Revalidar condições quando implementar. |

### Renda fixa

Identificar o produto antes de escolher o cálculo. Tesouro Selic tem título e vencimento próprios, quantidade e preços de referência publicados. Aplicação bancária indexada registra indexador, percentual/spread contratado, prazo, datas e convenções. Séries efetivas históricas podem apoiar uma estimativa do saldo bruto quando todas as regras forem conhecidas. A meta Selic serve como referência macroeconômica.

Distinguir saldo estimado, preço de mercado, valor líquido estimado de resgate e saldo informado por extrato. Custos e tributos dependem do produto e dos lotes/datas. O extrato permite conciliar diferenças e preservar ajustes identificados. O primeiro desenho exige acompanhamento gerencial; apuração tributária completa precisa de contrato próprio antes de implementação.

### Integração financeira, migração e exportação

Compra e venda vinculam a operação ao caixa efetivamente movimentado, com taxas e resultado discriminados. Aporte e retirada ligam as contas de origem/destino. Valorização de mercado atualiza a avaliação patrimonial. Consolidar caixa e posições uma única vez e tratar aportes, aquisição de ativos e rendimento conforme sua natureza nos relatórios.

A conversão do 1.x permite transformar contas usadas para MXRF11 ou outras aplicações em posições iniciais. Um saldo legado sozinho fornece somente o valor declarado. Quantidade e custo de aquisição precisam de dados comprováveis ou complemento manual. Identificar campos desconhecidos e preservar o histórico de origem sem fabricar compras ou rendimentos. O plano de migração revisa quais saldos deixam de representar caixa ao passar a representar ativos.

Exportação completa preserva instrumentos, custódias, movimentações, posições iniciais, preços utilizados, ajustes e vínculos financeiros. Todas as entidades pessoais conservam space_id e FKs, seguindo o contrato central. Instrumentos e posições têm identidades estáveis para mudanças de ticker e eventos corporativos.
