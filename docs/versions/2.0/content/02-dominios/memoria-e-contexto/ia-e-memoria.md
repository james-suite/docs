---
id: ia-e-memoria
title: IA para tags, agenda e documentos fiscais
description: Sugestões de tags, comandos delimitados para agenda e leitura fiscal por IA somente como fallback.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, tags, agenda-tarefas, diario, lugares-checkins, notificacoes, migrations, enums
source_refs: https://laravel.com/docs/13.x/ai-sdk
updated: 2026-10-28
---

## Direção de produto

No escopo planejado para o James 2.0, estão definidas duas funções: sugerir tags para transações e seus itens, podendo propor uma tag nova quando o catálogo não atender; e atuar como fallback na leitura de notas fiscais quando o XML, o QR ou um leitor determinístico disponível para a UF não resolver o documento. Uma tag nova só é criada após confirmação da pessoa. A IA não cria nem altera categorias financeiras.

Uma terceira função está em avaliação: interpretar pedidos explícitos de agenda em linguagem natural e preparar uma operação delimitada de criação, edição, cancelamento ou exclusão de compromisso. Isso não habilita um assistente conversacional geral, memória vetorial ou ações fora da agenda.

O modelo não confirma fatos financeiros. Valores, estabelecimento, produtos e outros campos extraídos da nota permanecem como candidatos para revisão. O modelo também não chama ferramentas de calendário diretamente: o servidor valida e executa somente as Actions permitidas, com autorização, escopo e auditoria.

## Chaves e provedores: recomendação inicial

Suportar dois modos explícitos: chave pessoal por usuário e provedor oferecido pelo administrador da instalação. IA começa desligada. O usuário escolhe uma conexão permitida e autoriza os tipos de dado enviados para aquela função. Chave pessoal é o padrão recomendado para instalações com várias pessoas, pois separa cobrança e quotas; uma chave da instalação é opcional e precisa de orçamento por usuário.

Credenciais pessoais ficam criptografadas no servidor, mascaradas na UI e fora de exportações, auditoria e logs. A chave de criptografia faz parte da recuperação da instalação. Essa proteção não equivale a criptografia ponta a ponta contra o administrador do servidor. Credencial global pode ficar no ambiente/cofre, com referência no banco e sem copiar o segredo para cada usuário.

Não alternar silenciosamente de provedor, chave ou modelo quando houver falha ou quota excedida. Uma conexão local pode ser suportada futuramente, mas endpoints arbitrários fornecidos por usuários exigem controle próprio de rede e não entram no primeiro formulário.

## Sugestão de tags para transações e itens

1. Aplicar regra local explícita por empresa/texto e opções já confirmadas.
2. Se solicitado, enviar só a descrição, o tipo de registro e o contexto mínimo necessário, com a lista limitada de tags daquele espaço.
3. Exigir resposta estruturada e validar novamente cada ID, limite de tamanho e pertença ao espaço. Nomes novos ficam separados das tags existentes e passam por verificação de duplicidade.
4. Registrar execução, provedor/modelo, versão do prompt, revisão do registro, uso e resultado sanitizado.
5. Mostrar sugestões para aceitar ou recusar. Após validação do modo automático, o usuário pode autorizar aplicação de tags existentes, com revisão/escopo rechecados e possibilidade de desfazer. Criar uma tag nova exige confirmação explícita; escolhas manuais não são substituídas.

O resultado do modelo não pode definir `space_id`, categoria financeira, executar SQL, chamar ferramentas financeiras nem disparar pagamentos. Descrições e comprovantes são conteúdo não confiável, inclusive se contiverem instruções para a IA.

## Fallback para documentos fiscais

A extração fiscal por IA é uma tarefa separada da classificação. Primeiro tentar XML/payload estruturado e um leitor determinístico disponível para a UF. Usar FiscalExtraction apenas quando esses caminhos não cobrirem o documento, com autorização específica antes de enviar nota/imagem à conexão escolhida. Aplicar orçamento e limites de dados da conexão; nenhuma falha permite trocar de provedor em silêncio.

A resposta estruturada preenche apenas candidatos com proveniência e confiança por campo. A pessoa revisa emitente/filial, itens, quantidades, descontos e totais antes da confirmação. O modelo não escolhe pagador, conta, veículo, produto correspondente ou visita como fato confirmado. Se a pessoa não autorizar IA ou o leitor falhar, a edição manual continua disponível.

## Comandos explícitos de agenda — proposta

Uma mensagem como “dia 2 de novembro às 14h tenho dentista” pode virar um rascunho tipado contendo ação, título, data/horário, fuso, duração e local somente quando informados ou resolvidos por preferências explícitas. Data incompleta deve ser interpretada com referência ao fuso e à data local atuais; se ainda houver mais de uma interpretação plausível, perguntar antes de executar.

Criação inequívoca pode ser salva sem uma etapa extra quando a pessoa habilitar essa preferência; exibir o resultado e oferecer desfazer. Edição, cancelamento e exclusão apresentam o evento-alvo e as mudanças, exigindo confirmação. Excluir usa lixeira reversível; cancelar preserva o compromisso com estado Cancelled. Se o evento mudar enquanto a IA prepara a proposta, marcar como desatualizada e recalcular após revisão.

A autorização dessa função é separada de classificação e extração fiscal. Enviar somente o texto necessário e, para localizar um alvo, os candidatos mínimos da agenda autorizada. Não enviar o calendário inteiro como contexto por padrão. Sem conexão permitida, orçamento ou aprovação de política, a pessoa pode continuar manualmente.

## Modos de aplicação

O [ClassificationMode](doc:enum-classification-mode) distingue Suggest e AutomaticExistingTags. A automação de tags é uma proposta opt-in depois de avaliar exemplos representativos; aplica somente tags do catálogo autorizado. O sistema registra qual política autorizou a aplicação e não inventa um revisor humano. Desfazer só remove atribuições daquela execução, preservando mudanças humanas posteriores. Essa política não autoriza salvar ou alterar eventos.

## Custo e falhas

Reservar orçamento antes da chamada; registrar consumo por usuário e conexão; liberar a reserva ou acertar o custo depois. Contagem de tokens/custo desconhecida deve aparecer como desconhecida/estimada, não como zero. Timeout, cancelamento e repetição precisam de idempotência. Não repetir automaticamente uma cobrança ambígua sem verificar o identificador da execução.

Ao mudar a transação ou seu catálogo, sugestões antigas passam a desatualizadas. Falha de IA não impede salvar dados nem escolher tags manualmente. Não guardar prompts e respostas completos por padrão quando bastarem hash, versão e resultado estruturado; definir retenção curta para diagnóstico opcional.

## Possibilidade futura: “JARVIS”

Uma evolução para assistente pessoal exigiria uma jornada e planejamento próprios. Não está incluída nas funções de IA previstas para o 2.0. A proposta de `records` e vínculos tipados pode organizar fontes reais, mas não autoriza por si só busca conversacional, embeddings ou memória derivada.

Se essa evolução for retomada, respostas precisarão apontar fontes e período; fatos extraídos guardarão referência, revisão e estado de confirmação. Apagar ou revogar uma fonte deverá invalidar resumos, caches e índices derivados.

## Tecnologia a avaliar em 2027

O Laravel AI SDK oferece uma interface para provedores e saída estruturada; é candidato a avaliar quando o projeto começar. O plano não fixa SDK, provedor ou modelo agora. Sua adoção deve permitir conexões por usuário e evitar configuração global mutável em workers compartilhados.

Referência: [Laravel AI SDK](https://laravel.com/docs/13.x/ai-sdk).
