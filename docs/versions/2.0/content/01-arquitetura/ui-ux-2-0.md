---
id: ui-ux-2-0
title: UI e UX — componentes e jornadas
description: Preservação da experiência atual com componentes Flux gratuitos e padrões do SGE.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: tags, cartoes, acertos, notificacoes, veiculos, produtos-precos, migracao-2-0, componentes-tecnicos, assistente-pessoal-rag
source_refs:
updated: 2026-11-07
---

## Linguagem do produto

Manter nomes e caminhos reconhecíveis: contatos, finanças, cartões, acertos e histórico. Melhorias devem reduzir trabalho para registrar a vida; não tornar cada cadastro uma ficha obrigatória extensa. Desktop é o foco de gestão; celular deve servir bem para captura, check-in, diário, recibo, gasto e consulta rápida.

Usar o starter kit disponível em 2027 e Flux 2 gratuito. Reaproveitar do SGE o select pesquisável/creatable, upload, loading, confirmação e componentes de auditoria, adaptando-os à identidade do James. O combobox é implementação própria inspirada no SGE; não pressupor licença Pro.

## Jornadas prioritárias

| Jornada | Comportamento proposto |
| --- | --- |
| Registrar compra | Descrição e valor primeiro; itens e vários pagamentos em painéis progressivos; mostrar valor pago e restante. |
| Escolher tags | Busca + múltiplas seleções + criar no próprio formulário; salvar nomes pendentes transacionalmente. Catálogo de conta nova começa vazio, sem lista padrão; tags descrevem o registro com precisão e evitam rótulos vagos. |
| Classificar com IA | Classificar separadamente as tags do cabeçalho e dos itens; aplicar ou descartar o conjunto no modo Suggest. Tags específicas ausentes são criadas junto com a aplicação, sem confirmação individual. Preservar escolhas manuais; falha da IA não bloqueia o formulário. Fallback fiscal também exige revisão dos campos extraídos. |
| Associar veículo | Vincular o veículo ao registro financeiro inteiro ou a itens determinados; exibir chip e filtro de veículo na área de tags, mantendo um vínculo tipado separado das tags. |
| Perguntar ao assistente pessoal (fase futura) | Responder com trechos e links para os registros usados, intervalo de tempo e ressalva de cobertura; permitir abrir/corrigir fonte e avaliar resposta. Ferramentas de escrita abrem uma proposta revisável pelo módulo dono do dado. |
| Pagar fatura | Histórico de pagamentos e contas de origem; distinguir fechamento, dívida, atraso e quitação. |
| Dividir despesa | Perguntar quem consumiu e quem pagou; “eu” opcional, parcela zero permitida. |
| Registrar acerto | Mostrar contato/empresa, valor em aberto, pagamento parcial e conta usada quando acompanhada; a liquidação atualiza o saldo sem criar receita/despesa nem uma tag de reembolso. |
| Analisar gastos por pessoa | Escolher a medida: consumo, valor adiantado, saldo a receber/pagar ou caixa recebido/pago. O contato/empresa é dimensão de relatório derivada dos vínculos; não precisa virar tag. |
| Marcar despesa para alguém | Associar a transação ou rateio à Party e definir consumo/pagador; um vínculo não presume dívida, e uma dívida não presume que houve movimentação na minha conta. |
| Arquivar instrumento | Mostrar saldo, faturas e recorrências afetadas; ocultar seletores usuais, manter histórico e quitações antigas. |
| Notificações | Pendências acionáveis e resumos; ler, arquivar ou adiar sem confundir com status de transporte. |
| Importar arquivo | Upload no navegador, progresso, prévia e confirmação para a própria conta, sem seletor de proprietário. |
| Serviço de veículo | Vários itens feitos, peças e mão de obra; tarefa pendente/checklist separado do serviço realizado. |
| Registrar odômetro | Ação curta na ficha do veículo, com valor, data/unidade e origem; não obriga criar abastecimento, serviço ou lançamento financeiro. |
| Ler nota em mercado/posto | Mostrar empresa, possível filial/endereço, itens, produtos sugeridos e confiança; pedir confirmação do lugar/produto quando houver homônimos ou campos incompletos. |
| Conferir preços | No detalhe do produto, mostrar preço por embalagem e por unidade-base, data e lugar; permitir montar uma cesta pessoal e explicar sua cobertura. |
| Diário/check-in | Formulário responsivo no próprio site, com seleção de lugar e botão opcional para usar a localização atual após permissão do navegador. |
| Histórico no mapa | Abrir eventos auditados num mapa privado, com filtro por módulo/período, agrupamento de pontos próximos e link para o registro. Atribuição OpenStreetMap permanece visível; mapa indisponível não afeta o histórico textual. |
| Localização ao salvar | Preferência por usuário, desligada inicialmente; com ela ativa e permissão do navegador, tentar capturar uma posição por submissão manual no site. Mostrar precisão, permitir “salvar sem localização” e nunca bloquear o registro por recusa, demora ou falha. A posição da importação, se capturada, pertence somente à ação de importar, não aos dados históricos importados. |

## Componentes próprios

TagPicker, CategoryPicker, AiTagClassifier, VehicleFinancialLinkPicker, PaymentEditor, SplitEditor, SettlementPaymentEditor, PartySettlementSummary, InvoicePaymentsHistory, ArchiveInstrumentDialog, NotificationInbox, ImportWizard, VehicleServiceEditor, ChecklistEditor, PrivateFileUpload e AuditHistory. Reusar componentes reais existentes antes de criar wrappers que apenas mudam nome de um componente Flux.

Todos têm teclado, foco visível, erro por campo, loading por ação e prevenção funcional de duplicatas no servidor. Seletores pesquisam somente dados da pessoa. Filtros e paginação sobrevivem à visita ao detalhe e retorno. Tabelas de gestão recebem versão móvel útil; o check-in funciona no navegador do celular sem aplicativo ou configuração externa.

## Seleção de contatos com Livewire

O PartyPicker é um componente próprio sobre a base gratuita escolhida em 2027 e o combobox do SGE. Compartilha busca, lista de resultados, teclado, seleção e criação rápida entre os módulos. Cada uso define o papel da Party, a cardinalidade e os tipos aceitos. A apresentação identifica pessoa/empresa, nome e detalhe de desambiguação disponível, como apelido ou identificação parcial permitida.

| Contexto | Seleção e retorno |
| --- | --- |
| Fornecedor de transação ou serviço | Uma pessoa/empresa e retorno à transação com dados preservados |
| Consumidores do rateio | Vários contatos com chips removíveis e tabela de cotas |
| Pagadores ao fornecedor | Seleção independente dos consumidores com valor por contribuição |
| Pagador de uma liquidação | Uma identidade, podendo diferir das contrapartes das obrigações |
| Titular/emissor de documento | Papel explicitado junto ao campo |
| Filtro de relatório | Referência tipada Party + papel + medida, com URL restaurável |

### Busca, criação e acessibilidade

Abrir o seletor mostra recentes da própria conta e a busca. Resultados limitados podem carregar mais, com busca digitada, atualização após pequena pausa e loading restrito ao componente. Preservar selecionados visíveis mesmo fora da página atual. Marcar arquivados nos vínculos históricos e oferecer filtro explícito quando a operação permitir usá-los.

A pessoa pode criar contato com nome e tipo no próprio fluxo. Campos adicionais ficam recolhidos. O novo contato aparece selecionado como pendente e é persistido junto ao formulário principal. Cancelamento descarta o rascunho. Usar identificador temporário estável para manter cotas e foco durante a criação. Homônimos legítimos devem poder coexistir e a tela deve permitir escolher após comparar os detalhes disponíveis.

Planejar navegação por setas, Enter, Escape e remoção de chip pelo teclado, com rótulo acessível e anúncio de carregamento/quantidade de resultados. No celular, o mesmo contrato pode abrir painel de seleção com busca focada e lista legível. Fechar devolve o foco ao campo de origem e mantém o rascunho completo.

As consultas Livewire recebem termo, papel e filtros permitidos. O servidor resolve espaço e autorização. Respostas de buscas anteriores devem ser descartadas quando o termo mudou. IDs selecionados e estado atual são revalidados no salvamento. Usar chaves estáveis por identidade, validação contextual, erro próximo da linha e bloqueio de envio duplicado durante a submissão. Alpine pode cuidar de abrir/fechar e teclado local conforme o componente escolhido.

### Rateio

O formulário exibe total da despesa, consumidores e contribuições de pagamento em seções próximas. A opção Eu pertence à identidade do espaço e pode ser marcada em cada papel. Consumir e pagar têm controles independentes. O total e o saldo a distribuir ficam visíveis enquanto as cotas são editadas.

Cada participante tem uma linha estável com identidade, participação no consumo, cota e contribuições. Após escolher contatos, permitir divisão igual entre consumidores selecionados ou edição exata dos valores. Preservar valores já digitados ao pesquisar ou adicionar outro contato. Alterar o grupo de consumidores recalcula somente após ação explícita de recalcular, com diferença visível antes de aplicar.

Antes de salvar, mostrar resumo por pessoa com cota, valor pago e saldo resultante. Para exemplo de R$ 120 pagos por mim para três contatos, mostrar Eu com consumo zero e adiantamento R$ 120, e três cotas de R$ 40. A criação de cotas e obrigações usa o plano de domínio revisado pelo servidor.

Remover uma pessoa com valores ou liquidações associadas exige apresentar as dependências e ação de correção. Reordenar linhas conserva identidades e quantias. Antes de salvar, divergências de soma aparecem junto ao total e às linhas envolvidas.

### Liquidar acertos

Escolher a contraparte mostra obrigações abertas, direção, moeda e saldos. Selecionar uma ou mais obrigações preenche o total sugerido e permite pagamento parcial. Um campo separado identifica quem efetivamente pagou. O resumo mostra destino de cada valor e saldo futuro. A confirmação persiste liquidação, alocações e pagamento financeiro opcional numa operação idempotente.

## Seletores de instrumentos e classificação

PaymentEditor mantém linhas independentes de valor, data, pagador e instrumento. Adicionar um segundo pagamento conserva o primeiro. Mostrar pago, agendado e saldo a pagar separados. A seleção de conta oferece o meio opcional em campo secundário. A seleção de cartão abre parcelas e primeira fatura prevista. Contas/cartões arquivados permanecem identificados em registros existentes, com seleção histórica acessível no fluxo apropriado.

CategoryPicker mostra o caminho Alimentação > Mercado. TagPicker aceita várias tags específicas. O seletor de filtros pode agrupar categorias, tags e relações a pessoas/veículos, com tipo e papel visíveis em cada chip.

FiscalAdjustmentEditor mostra desconto global, valor já atribuído e saldo restante. Escolher item e valor atualiza líquido conhecido e preço por unidade/peso. Distribuir o restante e desfazer uma atribuição são ações explícitas. O painel preserva os valores originais da nota para conferência.

DocumentPicker e seletores de veículos/produtos/lugares reutilizam comportamento de busca, seleção, estado vazio, criação e foco quando os contratos coincidirem. Cada domínio mantém suas regras e confirmação de dados específicos.

Na persistência de contato pendente, mapear seu identificador temporário ao Party criado pela operação idempotente. Repetir a mesma submissão reutiliza o mapeamento e os vínculos. A busca por nome oferece possíveis correspondências sem fundir homônimos automaticamente.

## Revisões compartilhadas

A prévia de correção mostra documentos, pagamentos, rateios, veículos e preços afetados antes de aplicar mudanças financeiras. Distinguir a versão de comprovante usada e a versão mais recente da biblioteca.

Na revisão de rateio parcialmente pago, mostrar consumo total, valor já financiado por pessoa e saldo do fornecedor. Ao selecionar pagamento em dinheiro, valor entregue e troco podem ajudar a calcular o líquido efetivo. Em cadastro manual usual, a opção de consumo pessoal aparece como sugestão visível e pode ser alterada. A importação usa a atribuição demonstrável ou identifica consumo desconhecido.

## Preparação da migração

ImportWizard oferece conversão inicial e envio opcional de arquivo preparado por ferramenta externa. A revisão compara origem e destino com agrupamentos de contatos, tags/categorias, compras/parcelas, acertos e documentos. A pessoa pode filtrar alterações, aceitar grupos, corrigir exceções e confirmar o plano completo em lote.

Mostrar dados financeiros originais, resultado esperado e diferenças justificadas em painéis próprios. A troca de arquivo ou edição do plano atualiza sua revisão, revalida dependências e pede novo aceite. Durante a publicação, o progresso acompanha somente a revisão já confirmada.

## Prototipação

Antes de congelar o schema, prototipar compra mista, fatura parcial, rateio sem parcela própria e importação com divergência. A interface deve explicar a diferença entre gasto, caixa, passivo e crédito de terceiros sem exigir conhecimento contábil.
