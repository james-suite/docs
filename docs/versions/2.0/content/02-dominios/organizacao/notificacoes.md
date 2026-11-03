---
id: notificacoes
title: Notificações e central de atenção
description: Eventos úteis, preferências pessoais, lembretes e histórico de tentativas de entrega.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, agenda-tarefas, ia-e-memoria, veiculos, migrations, enums
source_refs: https://github.com/james-suite/james/blob/master/app/Notifications/GeneralNotification.php, https://github.com/sge-suite/sge/blob/master/app/Models/EmailDeliveryAttempt.php, https://developer.apple.com/documentation/usernotifications/sending-web-push-notifications-in-web-apps-and-browsers, https://webkit.org/blog/13878/, https://web.dev/articles/push-notifications-faq, https://web.dev/articles/push-notifications-subscribing-a-user
updated: 2026-11-01
---

## Melhorar o que chega ao usuário

A central deve responder “o que precisa da minha atenção?” e permitir voltar ao registro, concluir, adiar ou dispensar quando fizer sentido. Separar aviso acionável, confirmação simples de operação e resumo periódico. Evitar notificar cada execução técnica bem-sucedida e repetir o mesmo vencimento em várias mensagens sem contexto.

Cada aviso tem tipo estável, versão do payload, gravidade, origem, destinatário, data e alvo por identificador seguro. Ler, arquivar e adiar são estados da experiência; sucesso/falha do e-mail ou Web Push pertence à entrega por canal.

## Fluxo proposto

Evento de domínio confirmado gera uma notificação por destinatário e uma intenção de entrega persistida. Workers processam as intenções após commit. Chave de deduplicação no banco considera destinatário, tipo, origem e ocorrência; cache pode acelerar, mas não é a garantia de não repetir.

Guardar tentativas de entrega separadas, inspiradas no SGE: destinatário efetivo, canal, mensagem/snapshot, número da tentativa, horário, status e erro sanitizado. Repetir o transporte não cria outra notificação. “Enviado” significa aceito pelo serviço de push ou e-mail; não significa que o sistema operacional exibiu nem que a pessoa leu o aviso.

## Preferências

Configuração por usuário e categoria: central interna, e-mail e Web Push; frequência imediata ou resumo; horário, fuso e quiet hours. Web Push é opcional por dispositivo e depende de permissão explícita. Nenhum canal externo é ativado sem escolha do usuário.

Avisos de segurança podem ter regra própria documentada. Notificações de finanças, manutenção e agenda devem permitir limitar repetição e período de antecedência.

## Lembretes que acompanham o domínio

Uma fatura paga deixa de gerar aviso de cobrança. Tarefa concluída e manutenção registrada recalculam seus lembretes. Uma notificação antiga aponta para o estado atual e explica quando a pendência já foi resolvida. Ações rápidas revalidam autorização, versão e estado; clicar duas vezes não executa a operação duas vezes.

Agrupar vencimentos próximos e resumos por período. Itens antigos podem ser arquivados sem apagar o histórico de entrega necessário. Importar dados não reenfileira notificações antigas ou mensagens para contatos externos.

## Web Push em web app instalado

Web Push é viável no iOS e no Android sem aplicativo nativo. O James precisa ser servido por HTTPS e registrar um Service Worker. No iOS/iPadOS 16.4 ou posterior, também precisa ser adicionado à Tela de Início como web app, com Web App Manifest em modo standalone/fullscreen; um site salvo apenas como favorito não basta. No Android, a instalação como PWA é opcional para receber push em navegadores compatíveis, embora dê acesso por ícone e uma experiência mais parecida com app. Verificar as APIs disponíveis no navegador em vez de presumir suporte pelo nome do dispositivo.

A pessoa ativa o canal dentro do James por um botão; o pedido de permissão e a criação da assinatura acontecem em resposta direta a esse toque. A assinatura é vinculada à conta e à instalação/dispositivo. Permitir mais de um dispositivo por conta, mostrar dispositivos ativos e oferecer revogação. O servidor usa Web Push padrão e VAPID; a chave privada VAPID é segredo de instalação e nunca vai ao navegador.

Notificações Push precisam ser visíveis; não depender de push silencioso para executar jobs. Se a assinatura expirar ou o serviço rejeitá-la, registrar a tentativa, desativar o endpoint inválido e manter o aviso na central. Push pode chegar com atraso ou não chegar se o aparelho estiver indisponível; a central no banco continua sendo a fonte de verdade.

O payload deve ser mínimo e não revelar valores financeiros, detalhes de saúde, texto do diário ou outros dados pessoais na tela bloqueada. Tocar em qualquer push abre sempre a tela `show` da própria notificação; o payload leva apenas seu identificador, nunca um link direto para a entidade de origem. Nessa tela, a pessoa vê o aviso e pode seguir uma ação para o registro relacionado. O servidor autentica e autoriza novamente a notificação e o alvo antes de mostrar os dados. Detalhes sensíveis ficam atrás do login.

No Android, navegadores compatíveis podem acordar o Service Worker para processar push mesmo quando a página não está aberta; a instalação PWA não é requisito geral. No iOS, Web Push está disponível para web apps adicionados à Tela de Início desde a versão 16.4. Em ambos os sistemas, a permissão parte de uma ação direta da pessoa e usa APIs padrão de Push, Notifications e Service Worker. Não exige app nativo nem inscrição no Apple Developer Program. Em navegadores/dispositivos sem suporte ou com permissão negada, a central interna continua funcionando e e-mail permanece uma opção independente. ([Apple Developer](https://developer.apple.com/documentation/usernotifications/sending-web-push-notifications-in-web-apps-and-browsers), [WebKit](https://webkit.org/blog/13878/), [web.dev: Android e Service Worker](https://web.dev/articles/push-notifications-faq), [web.dev: assinatura e detecção de suporte](https://web.dev/articles/push-notifications-subscribing-a-user))

## Interface e operação

Filtros por não lidas, pendências, categoria, canal e período; contagem consistente; detalhe com ação e motivo. Disponibilizar diagnóstico de falha de canal e reenvio autorizado. Não expor stack traces, tokens ou dados de outro usuário ao mostrar o erro.

Permissão negada, web app não instalado ou endpoint indisponível não bloqueiam a gravação do aviso. Tentar um transporte externo não pode bloquear a gravação do fato de negócio.
