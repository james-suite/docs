---
id: tags
title: Tags específicas, categorias e criação no combobox
description: Tags específicas em transações e itens, criadas no formulário ou pela classificação autorizada por IA.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, financas, ia-e-memoria, componentes-tecnicos, enums, migrations
source_refs: https://github.com/sge-suite/sge/blob/master/resources/views/components/select.blade.php, https://github.com/sge-suite/sge/blob/master/resources/views/components/select-create.blade.php, https://github.com/james-suite/james/blob/master/app/Models/FinancialTag.php
updated: 2026-11-07
---

## Categorias e tags têm papéis diferentes

Categoria financeira é uma classificação ampla para relatórios, como “alimentação” ou “transporte”. Tags devem identificar um contexto específico e reutilizável, como “gasolina aditivada”, “revisão dos 50 mil km”, “café em grãos — torra média” ou “viagem — novembro de 2026”. Evitar tags genéricas como “mercado”, “carro”, “compra”, “reembolso” ou “outros”; o detalhe amplo pertence à categoria ou ao módulo. A tag precisa ajudar a encontrar e entender registros semelhantes sem confundir coisas diferentes.

### Catálogo sem tags padrão

Um espaço novo começa com o catálogo de tags vazio. O James não instala, semeia ou sugere uma lista padrão de tags na criação da conta; não há tags globais compartilhadas entre usuários. As tags entram no catálogo quando a pessoa cria uma no formulário, quando importa as próprias tags do James 1.x ou quando uma classificação de IA autorizada precisa criar uma tag específica ausente. Categorias financeiras são um catálogo separado e não servem como tags iniciais.

Tags podem classificar transações, itens, diário, lugares, tarefas e documentos. Cada pessoa tem seu catálogo. Categorias podem ter hierarquia curta; suas relações precisam impedir ciclos. Tags têm nome normalizado, descrição, cor, ícone, aliases, arquivamento e fusão com histórico, além da origem de criação para distinguir inclusão manual, IA e importação. Pessoas, empresas, veículos, produtos, lugares e eventos mantêm vínculos tipados próprios; podem aparecer como chips/filtros e dimensões de gráfico, sem virar tags artificiais do catálogo.

## Combobox com criação no formulário

O SGE possui `select.blade.php` com semântica de combobox e o wrapper `select-create.blade.php`, que ativa busca e criação. Para o James, planejar um `TagPicker` com várias seleções, chips removíveis e criação inline, usando Flux 2 gratuito e o componente próprio como referência.

- Buscar somente no catálogo da pessoa autenticada e mostrar também aliases.
- Em catálogo vazio, mostrar o estado vazio e permitir criar uma tag específica; não popular opções padrão nem sugerir tags globais.
- Valores existentes trafegam por ID; nomes digitados ficam como seleções pendentes até o salvamento.
- Ao salvar o formulário, resolver/criar tags manuais na mesma transação. Cancelar o formulário não deve deixar tags órfãs.
- Uma restrição única por espaço e nome normalizado resolve criação concorrente. Se outra requisição criou o mesmo nome, reutilizar a tag existente.
- Não criar duplicatas apenas por caixa, espaços ou acento. Mostrar o nome encontrado antes de oferecer uma nova tag.
- Suportar teclado, foco, busca sem resultados, erros por campo e várias seleções. Não depender de componentes Flux Pro.

## Tags da transação e dos itens

Transações e itens recebem atribuições independentes por meio da mesma relação de atribuição. A transação sempre pode ter tags no cabeçalho, inclusive quando não tem itens; cada item pode ter tags próprias para detalhar aquela linha. Tags do cabeçalho descrevem a compra como um todo e não são copiadas para cada item. A interface pode mostrar o contexto combinado, identificando de qual nível veio cada tag, sem gravar atribuições duplicadas.

Busca e filtros podem consultar a união das tags diretas do cabeçalho e dos itens, mantendo o nível de origem para abrir o registro correto. Relatórios de gasto usam o valor uma única vez no nível apropriado: itens detalhados para relatórios por linha; cabeçalho para transações sem itens. Uma tag do cabeçalho encontrada por um item nunca replica nem multiplica o valor da compra.

A atribuição guarda origem (`manual`, `rule`, `ai`, `import`), responsável, revisão de origem e, quando existir, execução de IA. Atribuições manuais têm prioridade e não são substituídas por uma classificação posterior.

## Automação inicial

Regras locais explicáveis por empresa, texto, histórico e tipo de operação podem aplicar tags já existentes. Quando a pessoa executar uma classificação autorizada por IA, o modelo recebe o contexto mínimo do registro e o catálogo permitido. A classificação da transação e a de cada item são independentes; sem itens, só existe a classificação do cabeçalho. Se nenhuma tag existente descrever aquele registro com precisão, a IA propõe uma tag específica e cria a tag e sua atribuição na mesma aplicação autorizada; não há confirmação separada por tag. O sistema normaliza o nome, verifica nomes canônicos e aliases e reutiliza uma correspondência equivalente quando houver.

Tags criadas pela IA recebem origem, execução e auditoria. A pessoa pode revisar, renomear, fundir ou desfazer a aplicação depois; desfazer não remove uma tag que recebeu outros usos. Uma classificação não substitui tags manuais. Se o registro mudou durante a execução ou a IA não consegue formular uma tag realmente específica, não aplicar o resultado e mostrar o motivo. Categorias financeiras não são classificadas por IA. Não tratar a “confiança” declarada pelo modelo como probabilidade calibrada; correções podem alimentar regras locais sem treinar um modelo nem compartilhar dados entre pessoas.

## Migração

Preservar as tags e os vínculos que existam no cabeçalho e em cada item, sem projetar tags do cabeçalho sobre os itens. A tag primária pode gerar uma categoria correspondente quando o mapeamento for seguro; preservar também a tag e sua informação original para auditoria. Relatar colisões e qualquer fallback usado na prévia de migração.

As tags especiais atuais usam IDs fixos para reembolso, juros, saldo inicial, transferência, ajuste e pagamento parcial. No James, juros, saldo inicial, transferência, ajuste e pagamento pertencem ao tipo da operação; um valor devolvido por uma pessoa/empresa pertence a um acerto, e devolução de loja pertence ao fluxo comercial de Refund. Não criar uma tag “reembolso” para identificar essas operações nem continuar tomando decisões de saldo por ID de tag. Na migração, converter para relação de acerto somente quando pagamentos, obrigações e contraparte puderem ser conciliados; caso contrário, preservar o dado legado para revisão, sem inventar vínculo ou efeito contábil.

## Generalização para gráficos e detalhamento

Adotar categoria financeira hierárquica como generalização principal. Uma conta pode configurar Alimentação > Mercado e Alimentação > Restaurantes. Tags específicas acrescentam detalhes, como café em grãos e almoço de trabalho. A IA inicial continua dedicada a tags específicas e a pessoa pode usar categorias ou regras locais para a classificação ampla.

A categoria de cada item prevalece sobre a categoria de fallback da transação. Itens sem categoria usam o fallback, enquanto uma transação sem itens usa o cabeçalho. Somar uma categoria pai inclui seus descendentes uma vez. O mesmo registro categorizado em Mercado compõe Alimentação uma vez. Arquivar categoria conserva a hierarquia e os vínculos históricos.

Tags permanecem uma dimensão sobreposta. Uma linha pode conter duas tags específicas e participar de ambas as consultas. Um gráfico por tags deve informar sobreposição e oferecer total deduplicado da seleção. Gráficos de composição que precisam fechar em 100% usam categoria única por parcela de valor ou uma regra explícita de distribuição.

## Contatos e outros chips de contexto

A seleção unificada pode oferecer chips de Tag, Categoria, Pessoa/empresa, Veículo, Produto e Lugar. Cada opção guarda um par tipado kind/id e seu papel, por exemplo Party como fornecedor, consumidor, pagador ou contraparte do acerto. Mostrar ícone e legenda do tipo. Uma pessoa é uma dimensão virtual obtida da relação existente e pode agrupar filtros, linhas e gráficos.

Categorias e pessoas podem ter o mesmo nome de uma tag. Identidade tipada evita colisão e mantém navegação para a ficha correta. Renomear um contato atualiza sua apresentação por ID. Exportar kind, id e role, com os cadastros e relações necessários para resolver os nomes.

O filtro permite combinar papéis e medidas. Uma compra envolvendo Ana como consumidora e pagadora aparece uma vez no conjunto de transações filtradas. Para valores, consumo vem das cotas e adiantamento vem das contribuições/obrigações. O gráfico declara a medida escolhida, período, moeda e política de devoluções.

## Exportação analítica

Exportar categorias com parent_id, tags e aliases, assignments com nível transação/item, vínculos tipados e valores líquidos com cobertura de ajustes. A tela permite exportar o resultado filtrado em CSV e guardar metadados de filtro, medida e regra de agregação em arquivo acompanhante. O pacote integral de portabilidade inclui todas essas entidades e relações.

## Ajustes classificados e compatibilidade

FinancialAdjustment pode ter categoria e tags próprias, inclusive as tags de uma linha negativa importada. Sua medida é o valor assinado do ajuste. Ao agrupar tags de itens e ajustes, preservar a granularidade e indicar sobreposição quando a mesma seleção alcança líquido de item e desconto já alocado.
