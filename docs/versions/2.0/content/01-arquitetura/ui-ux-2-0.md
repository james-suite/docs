---
id: ui-ux-2-0
title: UI e UX — componentes e jornadas
description: Preservação da experiência atual com componentes Flux gratuitos e padrões do SGE.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: tags, cartoes, acertos, notificacoes, veiculos, produtos-precos, migracao-2-0, componentes-tecnicos
source_refs:
updated: 2026-10-26
---

## Linguagem do produto

Manter nomes e caminhos reconhecíveis: contatos, finanças, cartões, acertos e histórico. Melhorias devem reduzir trabalho para registrar a vida; não tornar cada cadastro uma ficha obrigatória extensa. Desktop é o foco de gestão; celular deve servir bem para captura, check-in, diário, recibo, gasto e consulta rápida.

Usar o starter kit disponível em 2027 e Flux 2 gratuito. Reaproveitar do SGE o select pesquisável/creatable, upload, loading, confirmação e componentes de auditoria, adaptando-os à identidade do James. O combobox é implementação própria inspirada no SGE; não pressupor licença Pro.

## Jornadas prioritárias

| Jornada | Comportamento proposto |
| --- | --- |
| Registrar compra | Descrição e valor primeiro; itens e vários pagamentos em painéis progressivos; mostrar valor pago e restante. |
| Escolher tags | Busca + múltiplas seleções + Criar no próprio formulário; salvar nomes pendentes transacionalmente. |
| Aceitar IA | Sugestões identificadas com aceitar/recusar; escolhas manuais preservadas e falha da IA não bloqueia formulário. |
| Pagar fatura | Histórico de pagamentos e contas de origem; distinguir fechamento, dívida, atraso e quitação. |
| Dividir despesa | Perguntar quem consumiu e quem pagou; “eu” opcional, parcela zero permitida. |
| Arquivar instrumento | Mostrar saldo, faturas e recorrências afetadas; ocultar seletores usuais, manter histórico e quitações antigas. |
| Notificações | Pendências acionáveis e resumos; ler, arquivar ou adiar sem confundir com status de transporte. |
| Importar arquivo | Upload no navegador, progresso, prévia e confirmação para a própria conta, sem seletor de proprietário. |
| Serviço de veículo | Vários itens feitos, peças e mão de obra; tarefa pendente/checklist separado do serviço realizado. |
| Registrar odômetro | Ação curta na ficha do veículo, com valor, data/unidade e origem; não obriga criar abastecimento, serviço ou lançamento financeiro. |
| Ler nota em mercado/posto | Mostrar empresa, possível filial/endereço, itens, produtos sugeridos e confiança; pedir confirmação do lugar/produto quando houver homônimos ou campos incompletos. |
| Conferir preços | No detalhe do produto, mostrar preço por embalagem e por unidade-base, data e lugar; permitir montar uma cesta pessoal e explicar sua cobertura. |
| Consultar estoque | Buscar item/código e ver saldo físico, reservado e disponível por local, lote, validade e condição; mostrar itens sem local atribuído. |
| Movimentar estoque | Receber, consumir/usar, transferir, contar, ajustar ou descartar; pedir só os dados relevantes à modalidade do item. Vínculo com finanças é opcional. |
| Repor | Sugerir quantidade entre saldo disponível, ponto mínimo e quantidade-alvo; editar lista e registrar recebimento em separado da compra financeira. |
| Diário/check-in | Formulário responsivo no próprio site, com seleção de lugar e botão opcional para usar a localização atual após permissão do navegador. |

## Componentes próprios

TagPicker, CategoryPicker, PaymentEditor, SplitEditor, InvoicePaymentsHistory, ArchiveInstrumentDialog, NotificationInbox, ImportWizard, VehicleServiceEditor, ChecklistEditor, PrivateFileUpload e AuditHistory. Reusar componentes reais existentes antes de criar wrappers que apenas mudam nome de um componente Flux.

Todos têm teclado, foco visível, erro por campo, loading por ação e prevenção funcional de duplicatas no servidor. Seletores pesquisam somente dados da pessoa. Filtros e paginação sobrevivem à visita ao detalhe e retorno. Tabelas de gestão recebem versão móvel útil; o check-in funciona no navegador do celular sem aplicativo ou configuração externa.

## Prototipação

Antes de congelar o schema, prototipar compra mista, fatura parcial, rateio sem parcela própria e importação com divergência. A interface deve explicar a diferença entre gasto, caixa, passivo e crédito de terceiros sem exigir conhecimento contábil.
