---
id: notificacoes
title: Notificações e central de atenção
description: Eventos úteis, preferências pessoais, lembretes e histórico de tentativas de entrega.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, agenda-tarefas, ia-e-memoria, veiculos, migrations, enums
source_refs: https://github.com/james-suite/james/blob/master/app/Notifications/GeneralNotification.php, https://github.com/sge-suite/sge/blob/master/app/Models/EmailDeliveryAttempt.php
updated: 2026-10-25
---

## Melhorar o que chega ao usuário

A central deve responder “o que precisa da minha atenção?” e permitir voltar ao registro, concluir, adiar ou dispensar quando fizer sentido. Separar aviso acionável, confirmação simples de operação e resumo periódico. Evitar notificar cada execução técnica bem-sucedida e repetir o mesmo vencimento em várias mensagens sem contexto.

Cada aviso tem tipo estável, versão do payload, gravidade, origem, destinatário, data e alvo por identificador seguro. Ler, arquivar e adiar são estados da experiência; sucesso/falha do e-mail ou Telegram pertence à entrega por canal.

## Fluxo proposto

Evento de domínio confirmado gera uma notificação por destinatário e uma intenção de entrega persistida. Workers processam as intenções após commit. Chave de deduplicação no banco considera destinatário, tipo, origem e ocorrência; cache pode acelerar, mas não é a garantia de não repetir.

Guardar tentativas de entrega separadas, inspiradas no SGE: destinatário efetivo, canal, mensagem/snapshot, número da tentativa, horário, status e erro sanitizado. Repetir o transporte não cria outra notificação. “Enviado” significa aceito pelo provedor; não significa lido pelo destinatário.

## Preferências

Configuração por usuário e categoria: canal interno, e-mail, Telegram; frequência imediata ou resumo; horário e fuso; quiet hours. Credenciais de canal devem ser pessoais ou explicitamente provisionadas para a instalação. Não reutilizar um `TELEGRAM_CHAT_ID` global para enviar os dados de todos.

Avisos de segurança podem ter regra própria documentada. Notificações de finanças, manutenção e agenda devem permitir limitar repetição e período de antecedência.

## Lembretes que acompanham o domínio

Uma fatura paga deixa de gerar aviso de cobrança. Tarefa concluída e manutenção registrada recalculam seus lembretes. Uma notificação antiga aponta para o estado atual e explica quando a pendência já foi resolvida. Ações rápidas revalidam autorização, versão e estado; clicar duas vezes não executa a operação duas vezes.

Agrupar vencimentos próximos e resumos por período. Itens antigos podem ser arquivados sem apagar o histórico de entrega necessário. Importar dados não reenfileira notificações antigas ou mensagens para contatos externos.

## Interface e operação

Filtros por não lidas, pendências, categoria, canal e período; contagem consistente; detalhe com ação e motivo. Disponibilizar diagnóstico de falha de canal e reenvio autorizado. Não expor stack traces, tokens ou dados de outro usuário ao mostrar o erro.

Web push e aplicativo são futuras opções de transporte, sem alterar o modelo central. Tentar um transporte externo não pode bloquear a gravação do fato de negócio.
