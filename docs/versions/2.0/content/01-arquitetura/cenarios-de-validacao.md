---
id: cenarios-de-validacao
title: Cenários de validação do desenho
description: Exemplos numéricos e comportamentos que o schema, a migração e as interfaces devem suportar.
type: acceptance-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, validação
related: planejamento-2-0, modelo-de-dados, financas, cartoes, acertos, tags, migracao-2-0, fases-2-0, assistente-pessoal-rag
source_refs:
updated: 2026-11-07
---

Estes casos são critérios para revisar o plano e, na implementação, criar testes de domínio e integração. Não representam testes já executados ou recursos implementados. Os números abaixo usam reais e nenhuma conversão de moeda.

## Financeiro e cartões

| Caso | Resultado esperado |
| --- | --- |
| Banco começa com R$ 1.000. Compra de R$ 150, sendo Pix de R$ 50 e cartão de R$ 100. | Uma transação de R$ 150; banco R$ 950; dívida do cartão R$ 100; consumo pessoal R$ 150. |
| Depois, pagar os R$ 100 da fatura com o mesmo banco. | Banco R$ 850 e dívida zero. Consumo continua R$ 150, sem outra despesa de R$ 100. |
| Fatura de R$ 500: pagamento de R$ 300 pela conta A e R$ 200 pela B, em dias diferentes. | Duas operações com datas preservadas, fatura quitada e cada conta debitada pelo seu valor. Quitar não reescreve todas as compras. |
| Pagamento de R$ 100 quita documento A de R$ 60 e B de R$ 40. Reversão técnica de R$ 20 alocada ao A. | A volta a ter R$ 20 a pagar; B continua quitado. A reversão aponta à alocação original e não pode ser aplicada de novo com a mesma chave. |
| Compra de R$ 100 no cartão com devolução comercial de R$ 20. | Compra permanece confirmada e paga ao fornecedor; documento de devolução ligado ao original reduz consumo líquido/dívida em R$ 20 conforme o fluxo. Não reabrir cobrança de R$ 20. |
| Tentar cancelar documento que ainda tem pagamento efetivo. | Recusar cancelamento. Exigir reversão técnica quando for correção; devolução comercial usa documento relacionado. |
| Arquivar conta com R$ 250 e cartão com R$ 90 de dívida. | Seletores usuais deixam de oferecê-los; histórico e patrimônio preservam valores. Ainda é possível quitar/estornar obrigações anteriores. |
| Recorrência mensal ancorada no dia 31. | Fevereiro ajusta para o último dia; março volta ao dia 31. Rodar o job duas vezes não duplica a ocorrência. |
| Transação legada de R$ 75 sem itens. | Um item de resumo identificado, total R$ 75, sem inventar produto/quantidade. Não falhar a invariante de soma nem apresentar detalhamento inexistente. |

## Rateios e classificação

| Caso | Resultado esperado |
| --- | --- |
| Usuário paga R$ 120 para três contatos; cada um consome R$ 40 e o usuário não consome. | Saída de caixa R$ 120, crédito a receber R$ 120 e despesa pessoal zero. Não dividir por quatro. |
| Ana devolve R$ 40 desse rateio. | Uma liquidação vinculada à obrigação da Ana reduz o saldo dela a zero; os outros R$ 80 continuam atribuídos aos outros contatos. Caixa aumenta R$ 40 se ligado a conta, receita pessoal permanece zero, e o gráfico por pessoa funciona sem tag “reembolso”. |
| Ana envia R$ 80 e, com registro explícito, quita sua obrigação de R$ 40 e parte da obrigação de R$ 40 do Bruno. | Uma liquidação registra Ana como pagadora e aloca os valores às duas obrigações; os saldos de Ana e Bruno atualizam separadamente. Não inferir que o pagador e os devedores são sempre a mesma pessoa. |
| Carlos paga uma compra compartilhada de R$ 80; eu consumi R$ 40 e ele R$ 40. Depois pago minha parte a Carlos pela conta bancária. | A compra original registra pagador Carlos; meu consumo é R$ 40 e minha obrigação a pagar é R$ 40. O pagamento de acerto reduz caixa e dívida, sem criar outra compra/despesa nem alterar quem consumiu. |
| A loja devolve R$ 40 de uma compra de R$ 120. | Documento de devolução e `PaymentKind::Refund` ligam o retorno à compra original; não criar `SettlementPayment` nem confundir com pagamento de contato. |
| Usuário só organiza um rateio entre contatos, sem pagar nem consumir. | Rateio e saldos entre contatos são consultáveis; nenhuma obrigação ou movimentação pessoal é criada. |
| Alterar um rateio que já possui liquidação. | Propor ajuste/revisão que preserve as obrigações liquidadas; não excluir e recriar os filhos antigos. |
| Uma transação tem dois itens de R$ 60 e R$ 40, mesma tag no cabeçalho e nos itens. | Busca e relatório dessa tag somam R$ 100, sem multiplicar os valores pela quantidade de vínculos. |
| Duas abas criam “Mercado” simultaneamente, uma com caixa/espaços diferentes. | Uma tag canônica, seleções apontando ao mesmo ID, sem erro bruto de unicidade. Cancelar um formulário não deixa tag nova órfã. |
| Tag canônica de uma pessoa coincide com alias de outra tag dela. | O namespace único encontra a tag existente ou exige resolução; não manter duas classificações ambíguas. |

## IA e notificações

| Caso | Resultado esperado |
| --- | --- |
| Resposta de IA chega depois de o usuário editar a transação. | Sugestão desatualizada, nenhuma aplicação silenciosa sobre a nova revisão. |
| Modelo devolve tag de outro espaço, ID inexistente ou instrução para mover dinheiro. | Resultado recusado; nenhuma alteração financeira ou acesso cruzado. |
| Busca futura do assistente pergunta por registro de outro usuário/espaço. | A consulta é filtrada/negada antes da montagem do contexto; nenhum título, trecho, embedding, cache ou citação cruzada é retornado. |
| RAG responde pergunta autobiográfica com uma nota e um serviço do veículo. | Cada afirmação verificável aponta para registro e data; conteúdo insuficiente resulta em resposta de não encontrado, não em fato inventado. |
| Pergunta “quanto gastei?” passa pelo assistente pessoal. | O modelo chama consulta financeira autorizada; soma, moeda e período vêm do domínio, e a resposta cita filtros usados sem somar trechos manualmente. |
| Texto recuperado contém instrução para ignorar regras ou revelar outro espaço. | Tratado como conteúdo não confiável; não muda instruções, escopo ou ferramentas disponíveis. |
| Fonte indexada é editada, removida ou excluída das permissões de IA. | Chunks, vetores, resumo e cache correspondentes são invalidados; revisão antiga não é usada para responder. |
| Índice HNSW retorna poucos candidatos depois do filtro por espaço/módulo. | Métrica compara recall com busca exata; ajusta-se o índice/consulta ou mantém busca exata, sem relaxar isolamento. |
| Chave pessoal falha e existe chave da instalação. | Falha identificada sem trocar de chave/provedor automaticamente. Salvar sem IA continua disponível. |
| Um endpoint Web Push expirou; há outro endpoint ativo para o mesmo usuário. | Registrar falha e desativar só a assinatura expirada; entregar ao endpoint ativo conforme a preferência e manter uma única notificação interna. |
| Job de vencimento é repetido enquanto a entrega Web Push falha uma vez. | Uma notificação, uma intenção por endpoint e tentativas distintas. Reprocessar transporte não duplica o aviso. |
| Usuário no iOS ainda não adicionou o James à Tela de Início ou negou a permissão de push. | Não criar assinatura nem insistir no prompt; avisos continuam na central e podem usar e-mail se estiver habilitado. |
| Usuário Android usa navegador compatível, ainda sem instalar o James como PWA, e aceita o push. | Registrar assinatura do navegador e entregar conforme preferência; a instalação PWA não é pré-requisito geral. |
| Pessoa toca em qualquer notificação push, inclusive quando o registro de origem já mudou de estado. | Abrir sempre o `show` da notificação; mostrar o estado atual e só então oferecer ação autorizada para o registro relacionado. |
| Fatura é paga antes do lembrete agendado. | Lembrete pendente é resolvido/cancelado; aviso antigo leva ao estado atual e não oferece cobrança já concluída. |

## Portabilidade e isolamento

| Caso | Resultado esperado |
| --- | --- |
| Usuário A envia pacote cujo usuário antigo tem o mesmo ID numérico de B no 2.0. | Dados importados somente para A; ID de origem não autoriza consulta ou gravação em B. |
| Reenviar o mesmo arquivo ou iniciar duas confirmações. | Reconhecimento de origem e exclusão mútua da importação; nenhuma duplicata. |
| Depois da prévia de destino vazio, criar uma transação antes de confirmar. | Prévia expira ou confirmação é bloqueada; nenhum dado pessoal novo é apagado para abrir espaço. |
| Uma linha antiga vira documento, pagamento e alocação; duas linhas de transferência viram uma operação. | Mapas suportam 1:N e N:1 por papel de conversão, mantendo referências e idempotência. |
| Compra em junho quitada em julho; o 1.x alterou o saldo na data da compra. | Conferir série mensal, mostrar diferença de regra temporal e preservar datas/proveniência. Saldo final igual não mascara mudança de relatório. |
| Pacote contém arquivo ausente, hash inválido, caminho ../ ou volume expandido além do limite. | Validação bloqueia a publicação e fornece diagnóstico; nenhum arquivo sai do staging privado. |

## Documentos e anexos

| Caso | Resultado esperado |
| --- | --- |
| Enviar um comprovante no formulário de transação. | Criar Document, primeira versão privada e vínculo à transação; o mesmo arquivo aparece na biblioteca sem novo upload. |
| Selecionar esse comprovante no serviço do veículo relacionado. | Criar somente outro vínculo ao mesmo Document; uma versão/original, com finanças como fonte de valores e veículo como fonte do serviço. |
| Remover o anexo da tela de transação quando ainda está vinculado ao veículo. | Remover apenas o vínculo com a transação; o documento continua na biblioteca e no serviço. Excluir o Document deve avisar sobre os vínculos restantes. |
| Anexar escritura ou certidão sem transação/bem correspondente. | Salvar na biblioteca como documento independente com campos opcionais; nenhuma relação financeira é inventada. |
| Enviar dois arquivos byte a byte iguais em transações diferentes. | Checksum pode avisar; manter dois documentos/vínculos até ação explícita da pessoa, sem mesclar os fatos automaticamente. |
| Importar `media` da coleção `attachments` do 1.x. | Criar uma versão e um vínculo por upload, preservar bytes/hash/ordem/origem e indicar ausências. Avatar de contato fica na coleção de perfil. |
| Usuário de outro espaço tenta consultar ou baixar documento por ID ou link conhecido. | Policy nega biblioteca, preview e original; possuir um vínculo ou URL assinada expirada não contorna o espaço proprietário. |
| Aviso de validade de uma certidão ou garantia chega por Web Push. | Abrir o `show` da notificação; de lá a ação abre o documento após nova autorização. |

## Veículos, diário e contexto

| Caso | Resultado esperado |
| --- | --- |
| Cadastrar um veículo e não criar um bem patrimonial. | Abastecimentos, serviços, custos e diário funcionam; ausência de bem não bloqueia o módulo. |
| Vincular esse veículo a um bem patrimonial e registrar uma avaliação. | O valor aparece uma única vez no patrimônio; o vínculo não cria custo, compra ou movimento de caixa. |
| Ler nota com um item de combustível, quantidade 40 L, preço R$ 6/L e desconto R$ 4. | Rascunho fiscal e abastecimento ligados; preço líquido R$ 5,90/L, transação conciliada e pagamento ainda depende de confirmar quem pagou. |
| Ler nota com combustível e outros itens, com desconto geral no total. | O scanner não atribui o desconto inteiro ao combustível nem inventa rateio; pede alocação ou deixa o preço líquido por litro sem confirmação. |
| Confirmar nota de abastecimento paga por um contato. | Custo e pagador ficam registrados; nenhuma conta bancária pessoal é debitada e nenhuma dívida é criada sem acerto explícito. |
| Comprar óleo para um veículo em uma data e trocar semanas depois. | Compra ligada ao veículo sem serviço concluído na primeira data; evento de troca usa a data/quilometragem real e pode referenciar o item comprado. |
| Registrar “ouvi um ruído ao frear” no diário do veículo. | Nota aparece na linha do tempo, sem criar despesa, manutenção concluída ou mudança de odômetro. |
| Oficina realiza óleo R$ 80 e mão de obra R$ 40 em uma visita. | Um evento com dois itens e despesa total R$ 120 ligada a uma transação, sem gerar R$ 240 ao consultar pelos dois módulos. |
| Uma oficina de R$ 120 é ligada ao veículo e dividida em R$ 60 para mim e R$ 60 para outra pessoa. | Custo do veículo continua R$ 120; consumo pessoal é R$ 60; obrigação da contraparte é R$ 60. O gráfico por veículo e o gráfico por pessoa usam o mesmo fato sem duplicar a despesa. |
| Tanque cheio no km 1.000; parcial de 5 L no km 1.100; cheio com 15 L no km 1.250. | Intervalo completo considera 250 km e 20 L, resultando 12,5 km/L; não usar só os últimos 15 L nem o volume do abastecimento inicial. |
| Marcar checklist de manutenção concluído sem registrar o serviço. | Checklist muda de estado, mas odômetro/base do plano não são atualizados como se o serviço tivesse ocorrido. |
| Registrar 82.450 km na ficha do veículo sem abastecer nem pagar nada. | Uma leitura independente aparece na linha do tempo e atualiza as consultas futuras de distância; não cria evento de abastecimento ou transação. |
| Compra numa empresa com endereço conhecido. | Pode sugerir associação ao lugar; não criar visita confirmada automaticamente. |
| Fazer check-in pelo site sem informar coordenadas. | Criar a visita com lugar e horário, sem exigir permissão de localização. |
| Tocar em “Usar minha localização” e autorizar o navegador. | Uma posição e sua precisão são associadas somente ao check-in em andamento. |
| Negar a permissão ou não obter posição no navegador. | Mostrar uma explicação breve e permitir salvar o check-in com lugar selecionado ou preenchido manualmente. |
| Ativar captura ao salvar e criar ou editar um registro no site. | Uma posição pontual é associada ao evento por correlation_id; contexto, precisão e horário aparecem no histórico do registro. |
| Uma gravação cria uma transação, itens e alocações com vários eventos de auditoria. | Todos usam o mesmo contexto de localização da ação; não há cópias por item nem várias posições. |
| Negar localização, exceder o tempo limite, usar navegador sem suporte ou falhar o serviço de mapas. | Salvar e consultar o registro/histórico textual normalmente; não usar IP como substituto nem criar ponto sem coordenadas. |
| Job de recorrência, processamento assíncrono de importação ou execução de IA altera registros. | Auditoria identifica origem automática sem atribuir a posição atual do navegador ou de uma ação humana anterior. |
| A pessoa confirma uma importação no site com captura de localização habilitada e permissão concedida. | Um contexto pode registrar a posição daquela confirmação; não copiar coordenadas para os registros históricos importados. |
| Abrir mapa com tiles OpenStreetMap. | Exibir atribuição visível, filtros e agrupamento; mapa mostra precisão reportada e não envia metadados do registro ao provedor. |
| Expirar/apagar contexto geográfico associado a um evento. | Remover a posição do mapa mantendo ação, autoria, data e diff funcional no Activity Log. |
| Editar texto íntimo do diário. | Revisão protegida no diário; Activity Log registra autoria/evento sem copiar todo o texto para diffs comuns. |

## Produtos, preços e captura fiscal

| Caso | Resultado esperado |
| --- | --- |
| Nota de mercado tem três pacotes iguais de 500 g, GTIN conhecido e valor líquido total de R$ 24. | Revisar/associar ao produto; registrar quantidade-base de 1,5 kg, preço de R$ 8 por pacote e R$ 16/kg, ligado uma única vez à linha financeira. |
| Duas compras do mesmo GTIN, uma em cada mercado e em meses diferentes. | Histórico compara preço por unidade-base, data e lugar sem duplicar despesas; texto original de cada nota continua preservado. |
| Nota de vários produtos tem desconto geral de R$ 10 sem rateio por item. | Não atribuir os R$ 10 a um produto específico nem publicar preço líquido unitário como confirmado até revisar a alocação. |
| Cesta pessoal contém quatro produtos, mas um não tem preço no período atual. | Mostrar o índice do subconjunto comparável e a cobertura; marcar como parcial. Não inventar ou interpolar o preço ausente. |
| QR identifica empresa/endereço fiscal de um posto, mas ainda não há coordenadas. | Sugerir ou resolver a filial pelo identificador/endereço; deixar coordenadas vazias e não criar visita/check-in. |
| UF não tem leitor determinístico disponível e a conexão fiscal de IA não foi autorizada. | Não enviar o documento a modelo; manter revisão manual e salvar normalmente. |
| Leitor determinístico falha e usuário permite FiscalExtraction. | Criar candidatos com processador/versão e confiança disponíveis; exigir revisão humana e não confirmar pagamento, pagador, produto ou filial automaticamente. |

## Casos complementares de financeiro e seleção

| Caso | Resultado esperado |
| --- | --- |
| Compra de R$ 300 paga com R$ 50 em dinheiro, R$ 70 Pix e R$ 180 em três parcelas no cartão. | Uma compra, três pagamentos e charge de R$ 180 com três parcelas de R$ 60. Consumo R$ 300, caixa sai R$ 120 e dívida cresce R$ 180. |
| Nota de R$ 150 brutos tem R$ 15 de desconto global. Atribuir R$ 10 à carne de 2 kg por R$ 80. | Líquido conhecido da carne R$ 70 e R$ 35/kg, desconto residual R$ 5 e total da compra R$ 135. Preço exibe cobertura pendente até resolver o ajuste aplicável. |
| Cabeçalho da nota resume R$ 15 de descontos já discriminados nos itens. | Registrar o resumo fiscal sem criar desconto adicional. Total preservado e procedência disponível. |
| Atribuir R$ 10 do mesmo ajuste em duas abas quando só restam R$ 10. | Uma atribuição válida e a outra recusada/revisada sob bloqueio, com resíduo sempre não negativo. |
| Mercado é filha de Alimentação e um item tem categoria Mercado. | Alimentação inclui o item uma vez. Tags sobrepostas têm total deduplicado e aviso de sobreposição quando agrupadas. |
| Ana aparece como pagadora e consumidora e também existe uma tag com o nome Ana. | Seleção preserva kind/id/role. Filtro Party usa suas relações e medida escolhida, e o filtro Tag usa assignments próprios. |
| Arquivar cartão com parcelas futuras e depois pagar fatura antiga. | Preservar parcelas, dívida e quitação acessível. Compra nova usa instrumento ativo ou exige reativação. |
| Conta Wallet importada com saldo conhecido. | Tipo wallet e saldo preservados, IDs remapeados e rótulo Dinheiro físico. |
| Buscar contato, digitar cota e receber resposta atrasada de busca anterior no Livewire. | Lista atual permanece coerente e valor/cota/foco ficam preservados por identidade estável. |
| Criar contato pendente no rateio e cancelar o formulário. | Rascunho e cotas pendentes são descartados juntos. Homônimo existente pode ser escolhido ou uma pessoa distinta pode ser criada explicitamente. |
| Eu pago R$ 120 para três contatos selecionados e deixo meu consumo em zero. | Resumo apresenta três cotas de R$ 40 e meu adiantamento de R$ 120 antes da confirmação. |
| Exportar conta com instrumentos arquivados, tags, documentos versionados e ajustes parciais. | Manifesto cobre todos os domínios retidos, relações e arquivos. JSON preserva revisões e resíduo, e relatório pode reproduzir o total. |
| Novo módulo adiciona tabela pessoal sem exportador. | Checklist de entrega e finalização do pacote completo indicam cobertura incompleta até registrar o exportador. |
| Restaurar backup de toda a instalação em ambiente isolado. | Banco, arquivos, chaves e configuração recuperáveis, com contagens/hashes/saldos verificados antes de liberar acesso e entregas externas. |

## Correções e integrações da revisão completa

| Caso | Resultado esperado |
| --- | --- |
| Compra de R$ 120 para Ana e Bruno, só R$ 60 pagos por mim, R$ 30 financiados para cada um. | Dois créditos de R$ 30 e R$ 60 pendentes ao fornecedor. Cotas totais continuam R$ 60 por pessoa. |
| Pagar R$ 120 e definir depois que o consumo inteiro era dos contatos. | Reclassificação de contrapartida cria o crédito a receber, com saldo de caixa preservado e operação idempotente. |
| Corrigir R$ 15 de uma liquidação de R$ 40 recebida em conta. | Reabrir R$ 15 da obrigação e reverter R$ 15 no instrumento original na mesma operação. |
| Devolver R$ 100 por Pix de compra paga com R$ 60 Pix e R$ 40 cartão. | Um recebimento de R$ 100 e duas referências às alocações originais. O crédito em conta é R$ 100 e a dívida do cartão segue até a quitação. |
| Corrigir desconto de item já ligado a veículo, preço e rateio. | Conservar total/pagamentos e validar alocações afetadas. Preços recebem nova revisão e consumo detalhado informa sua cobertura. |
| Nota 1.x tem linhas de R$ 80 e R$ 20, mais ajuste de preço negativo totalizando menos R$ 10. | Dois itens usuais e um FinancialAdjustment Discount de R$ 10. Compra total R$ 90, com tags e valores originais rastreáveis no mapa. |
| Adicionar nova digitalização a documento usado como comprovante de compra. | A compra abre a versão fixada e a biblioteca oferece a nova versão. |
| Duas abas enviam versão nova do mesmo documento. | Números distintos sob bloqueio e unicidade, com cada upload idempotente e bytes publicados uma vez. |
| Worker cai após preparar arquivo, antes de publicar os vínculos. | Versão pendente recuperável e limpeza respeitando lease, referências e publicação. |
| Corrigir observação de preço da mesma linha. | Uma corrente e revisões históricas preservadas, sem colisão da unicidade nem duplicação na cesta. |
| Apagar lote de saúde que compartilha amostra com outro lote. | Mostrar a associação compartilhada e preservar a amostra conforme seus vínculos/revisões confirmados. |
| Habilitar agenda manual sem IA e tarefas sem veículos. | Migrations mínimas e operações funcionam, com extensões opcionais aplicadas somente quando suas pontas existirem. |
| Criar evento de dia inteiro em 2 de novembro. | Início 2 e fim exclusivo 3, exibido como um dia no fuso local. |
| Editar horário e depois cancelar evento publicado no feed. | Identidade estável e revisões atualizadas, com comportamento de cancelamento validado no cliente e tombstone retido pelo prazo definido. |
| Sair da conta A e entrar em B no mesmo navegador com Web Push. | Revogar associação de A, confirmar a de B e revalidar o destinatário antes de cada envio. Um aviso antigo exige a conta proprietária no show. |
| Exportação completa fixa versão de arquivo enquanto uma limpeza tenta removê-la. | Referência temporária conserva o original ou a exportação falha com diagnóstico antes de publicar pacote completo. |
| Worker de importação antigo retorna após outro recuperar a operação. | Geração de lease divergente impede publicação pelo worker antigo. |
| Categoria de uma linha, tag, pessoa e veículo chegam ao mesmo filtro de relatório. | Identidades e papéis tipados, deduplicação do conjunto e medida explicitada antes da soma. |
| Mudar o fuso padrão após criar recorrência. | A recorrência mantém seu fuso/âncora até edição explícita e preserva chaves das ocorrências já geradas. |
| Nome de produto começa com expressão interpretável como fórmula na planilha. | CSV usa escape documentado e JSON mantém o valor original. |

## Casos de completude do schema

| Caso | Resultado esperado |
| --- | --- |
| Um PDF contém duas notas fiscais distintas e depois recebe nova digitalização. | Cada nota referencia sua origem de versão e identidade fiscal. As extrações anteriores preservam os bytes usados. |
| Um serviço recebe leitura de odômetro de outro veículo do mesmo usuário. | Validação recusa a referência incompatível mesmo com space_id igual. |
| Modelo de tarefa é usado com o módulo de veículos desabilitado. | Tarefa e checklist podem ser salvos com o contrato genérico. |
| Compra de R$ 150 recebe R$ 200 em espécie e devolve R$ 50 de troco. | Pagamento líquido e redução da conta Dinheiro físico são R$ 150. Dados de entregue/troco continuam rastreáveis. |
| Duas execuções usam a chave geral de IA com limites globais quase esgotados. | Reserva pessoal e global é coordenada sob bloqueio e somente chamadas cobertas são autorizadas. |
| Correção de preço chega depois de uma consulta que fixou snapshot anterior. | Consulta histórica mantém sua revisão e nova consulta pode usar a substituta corrente. |
| Importar pacote pessoal cujo módulo de documentos está ausente no destino. | Prévia indica dependência e bloqueia publicação completa até disponibilizar o contrato necessário. |
| Cesta contém preço zero conhecido de promoção em um período. | Preservar zero como observado, exigir custo-base total positivo e declarar cobertura/política da cesta. |

## Gate de aprovação

O protótipo de schema precisa resolver os casos centrais de dinheiro, cartão, rateio e migração antes de consolidar ledger/records. Os demais casos orientam cada marco; não exigir todos os módulos futuros para validar a migração do núcleo atual.
