---
id: ia-e-memoria
title: IA e memória pessoal do James
description: Assistência simples com tags agora e uma base rastreável para o assistente pessoal futuro.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, tags, diario, lugares-checkins, notificacoes, migrations, enums
source_refs: https://laravel.com/docs/13.x/ai-sdk
updated: 2026-10-26
---

## Direção de produto

Evoluir o James para um assistente que encontre relações e responda sobre a vida do próprio usuário. A fundação é guardar dados corretos, datas, fontes, vínculos e permissões. A primeira entrega prioriza classificação, com sugestões por padrão e aplicação opcional de tags existentes. A extração fiscal por IA é um uso separado e restrito, somente como fallback autorizado quando os leitores determinísticos não atenderem. Interface de conversa, memória vetorial e ações autônomas pertencem a fases futuras.

Registrar mais dados só ajuda quando cada dado tem significado e pode ser corrigido, exportado e apagado. O registro de uma compra não prova uma visita; uma sugestão de IA não prova um fato biográfico; ausência de dados não significa que um evento não aconteceu.

## Chaves e provedores: recomendação inicial

Suportar dois modos explícitos: chave pessoal por usuário e provedor oferecido pelo administrador da instalação. IA começa desligada. O usuário escolhe uma conexão permitida e autoriza os tipos de dado enviados para aquela função. Chave pessoal é o padrão recomendado para instalações com várias pessoas, pois separa cobrança e quotas; uma chave da instalação é opcional e precisa de orçamento por usuário.

Credenciais pessoais ficam criptografadas no servidor, mascaradas na UI e fora de exportações, auditoria e logs. A chave de criptografia faz parte da recuperação da instalação. Essa proteção não equivale a criptografia ponta a ponta contra o administrador do servidor. Credencial global pode ficar no ambiente/cofre, com referência no banco e sem copiar o segredo para cada usuário.

Não alternar silenciosamente de provedor, chave ou modelo quando houver falha ou quota excedida. Uma conexão local pode ser suportada futuramente, mas endpoints arbitrários fornecidos por usuários exigem controle próprio de rede e não entram no primeiro formulário.

## Pipeline inicial de classificação

1. Aplicar regra local explícita por empresa/texto e opções já confirmadas.
2. Se solicitado, enviar só a descrição e o contexto mínimo necessário, com lista limitada de tags/categorias daquele espaço.
3. Exigir resposta estruturada e validar novamente cada ID, limite de tamanho e pertença ao espaço.
4. Registrar execução, provedor/modelo, versão do prompt, revisão do registro, uso e resultado sanitizado.
5. Mostrar sugestão para aceitar ou recusar. Após validação do modo automático, o usuário pode autorizar aplicação de tags existentes, com revisão/escopo rechecados e possibilidade de desfazer. Categoria e nome novo exigem revisão; escolhas manuais não são substituídas.

O resultado do modelo não pode definir `space_id`, executar SQL, chamar ferramentas financeiras nem disparar pagamentos. Descrições e comprovantes são conteúdo não confiável, inclusive se contiverem instruções para a IA.

## Fallback para documentos fiscais

A extração fiscal por IA é uma tarefa separada da classificação. Primeiro tentar XML/payload estruturado e um leitor determinístico disponível para a UF. Usar FiscalExtraction apenas quando esses caminhos não cobrirem o documento, com autorização específica antes de enviar nota/imagem à conexão escolhida. Aplicar orçamento e limites de dados da conexão; nenhuma falha permite trocar de provedor em silêncio.

A resposta estruturada preenche apenas candidatos com proveniência e confiança por campo. A pessoa revisa emitente/filial, itens, quantidades, descontos e totais antes da confirmação. O modelo não escolhe pagador, conta, veículo, produto correspondente ou visita como fato confirmado. Se a pessoa não autorizar IA ou o leitor falhar, a edição manual continua disponível.

## Modos de aplicação

O [ClassificationMode](doc:enum-classification-mode) distingue Suggest e AutomaticExistingTags. A automação é uma proposta opt-in depois de avaliar exemplos representativos; aplica somente tags do catálogo autorizado. O sistema registra qual política autorizou a aplicação e não inventa um revisor humano. Desfazer só remove atribuições daquela execução, preservando mudanças humanas posteriores.

## Custo e falhas

Reservar orçamento antes da chamada; registrar consumo por usuário e conexão; liberar a reserva ou acertar o custo depois. Contagem de tokens/custo desconhecida deve aparecer como desconhecida/estimada, não como zero. Timeout, cancelamento e repetição precisam de idempotência. Não repetir automaticamente uma cobrança ambígua sem verificar o identificador da execução.

Ao mudar a transação ou seu catálogo, sugestões antigas passam a desatualizadas. Falha de IA não impede salvar dados nem escolher tags manualmente. Não guardar prompts e respostas completos por padrão quando bastarem hash, versão e resultado estruturado; definir retenção curta para diagnóstico opcional.

## Base para o futuro “JARVIS”

A proposta de `records` e vínculos tipados organiza fontes reais: pessoa, empresa, compra, evento de veículo, diário, lugar e compromisso. A busca começa textual e por filtros. Embeddings, conversas e memória derivada só entram quando houver uma jornada validada, com índices que preservem o mesmo escopo e exclusão dos dados originais.

Uma resposta futura precisa apontar fontes e período. Fatos extraídos guardam referência, revisão e estado de confirmação; não substituir o registro original por um resumo. Diário, saúde e localização ficam fora da IA por padrão. Apagar/revogar uma fonte deve invalidar resumos, caches e índices derivados.

## Tecnologia a avaliar em 2027

O Laravel AI SDK oferece uma interface para provedores e saída estruturada; é candidato a avaliar quando o projeto começar. O plano não fixa SDK, provedor ou modelo agora. Sua adoção deve permitir conexões por usuário e evitar configuração global mutável em workers compartilhados.

Referência: [Laravel AI SDK](https://laravel.com/docs/13.x/ai-sdk).
