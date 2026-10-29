---
id: agenda-tarefas
title: Agenda, tarefas e lembretes
description: Base compartilhada entre diário, veículos, finanças e compromissos pessoais.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, notificacoes, veiculos, diario, migrations
source_refs: https://brasilapi.com.br/docs
updated: 2026-10-28
---

## Contrato

Compromissos guardam início/fim, fuso original e sinal de dia inteiro. Tarefas guardam estado, prazo opcional, prioridade opcional e vínculo a um registro. Checklist é uma sequência de itens com conclusão e autoria; não confundir tarefa concluída com um fato financeiro pago.

Lembretes pertencem a um alvo e têm ocorrência identificável. Na entrega inicial, suportar data/horário e regras simples de repetição. Regras de manutenção por km usam o domínio de veículos; agenda apenas apresenta o resultado.

## Recorrência e UX

Preservar âncora e fuso da série. Editar uma ocorrência não modifica todas; oferecer “esta ocorrência” e “as próximas” com significado claro. Eventos de dia inteiro usam datas, não conversões UTC que mudem o dia.

Uma tarefa pode estar vinculada a diário, veículo, pessoa ou compra. O vínculo leva ao registro original. Mostrar pendências consolidadas na home sem exigir que cada módulo gere avisos idênticos.

## Limites iniciais

Não planejar colaboração entre usuários, convites a terceiros ou sincronização bidirecional de calendários antes de requisitos próprios. Importação/exportação ICS e integração externa são posteriores à consistência da agenda local.

O calendário pode oferecer uma camada visual de feriados nacionais e estaduais consultados por ano/UF. A fonte BrasilAPI não cobre todos os feriados municipais; não apresentar a lista como calendário legal completo nem deslocar vencimentos, tarefas ou compromissos automaticamente. Se a consulta estiver indisponível, os eventos pessoais continuam funcionando.
