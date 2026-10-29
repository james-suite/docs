---
id: agenda-tarefas
title: Agenda, tarefas e lembretes
description: Base compartilhada entre diário, veículos, finanças e compromissos pessoais.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, notificacoes, veiculos, diario, migrations, ia-e-memoria
source_refs: https://support.apple.com/guide/iphone/iphc37be2016/27/ios, https://support.apple.com/guide/iphone/iph3d1110d4/27/ios, https://developers.google.com/workspace/calendar/api/v3/reference/events
updated: 2026-10-28
---

## Contrato

Compromissos guardam início/fim, fuso original e sinal de dia inteiro. Tarefas guardam estado, prazo opcional, prioridade opcional e vínculo a um registro. Checklist é uma sequência de itens com conclusão e autoria; não confundir tarefa concluída com um fato financeiro pago.

Lembretes pertencem a um alvo e têm ocorrência identificável. Na entrega inicial, suportar data/horário e regras simples de repetição. Regras de manutenção por km usam o domínio de veículos; agenda apenas apresenta o resultado.

## Recorrência e UX

Preservar âncora e fuso da série. Editar uma ocorrência não modifica todas; oferecer “esta ocorrência” e “as próximas” com significado claro. Eventos de dia inteiro usam datas, não conversões UTC que mudem o dia.

Uma tarefa pode estar vinculada a diário, veículo, pessoa ou compra. O vínculo leva ao registro original. Mostrar pendências consolidadas na home sem exigir que cada módulo gere avisos idênticos.

## Comandos de agenda por linguagem natural

Como proposta para o 2.0, a pessoa pode escrever “dia 2 de novembro às 14h tenho dentista” e pedir para criar, editar, cancelar ou excluir um compromisso. A função de IA é interpretar a solicitação e devolver uma operação tipada para a agenda; ela não recebe acesso direto ao banco nem pode executar ações fora desse conjunto.

Para criar, interpretar datas no fuso do espaço e resolver ano ausente pela próxima ocorrência futura; mostrar sempre a data completa. Não inventar duração ou local. O James pode salvar automaticamente criações inequívocas se essa preferência estiver habilitada, oferecendo desfazer. Quando houver ambiguidade — como duas ocorrências de “dentista” — apresentar as opções e pedir que a pessoa escolha.

Para editar ou excluir, localizar apenas eventos do espaço autorizado, mostrar qual evento será alterado e apresentar as mudanças propostas. Exigir confirmação antes da gravação; usar exclusão reversível e registrar a ação na auditoria. Cancelar mantém o evento no histórico com estado `cancelled`; excluir o move para a lixeira. A proposta expira ou fica desatualizada se a revisão do evento mudar antes da confirmação.

Falha ou desativação da IA não bloqueia a agenda manual. Título, data, horário, fuso, duração, local e recorrência continuam editáveis no formulário convencional. Enviar ao provedor somente a solicitação e o contexto mínimo necessários, após autorização da função de agenda.

## Calendários externos: iPhone e Google

O James mantém sua própria agenda como fonte de verdade. “Calendário do iPhone” é o aplicativo que pode mostrar contas iCloud, Google e CalDAV; não existe uma API web do iOS que o site Laravel possa chamar diretamente. O iPhone permite adicionar uma conta CalDAV e assinar um calendário externo `.ics` somente para leitura.

| Caminho | O que oferece | Limite principal |
| --- | --- | --- |
| Arquivo `.ics` por evento | Abrir/importar um evento pontual no Calendário do iPhone. | Importação avulsa; alterações posteriores no James não acompanham o evento. |
| Feed `.ics` por URL secreta | Ver e receber atualizações da agenda James no iPhone por assinatura. | Leitura unidirecional; alterações feitas no iPhone não retornam ao James. Qualquer pessoa que obtenha a URL pode ler o feed até a revogação. |
| Conta CalDAV do James | Adicionar o próprio James em Ajustes > Apps > Calendário > Contas de Calendário e editar/sincronizar eventos pelo app Calendário do iPhone. | A agenda aparece como uma conta separada ao lado do iCloud. Exige implementar e manter o protocolo CalDAV, autenticação, concorrência e compatibilidade dos clientes. |
| Google Calendar API | Conectar um calendário Google por OAuth para criar, atualizar, remover e sincronizar eventos. | Requer consentimento OAuth, escopos mínimos, armazenamento seguro de credenciais e resolução de conflitos. |

O feed `.ics` escolhido para a primeira integração não escreve no calendário iCloud nem traz alterações do iPhone de volta ao James. Em uma fase posterior, CalDAV e Google podem oferecer sincronização bidirecional: o evento salvo no James seria enviado às conexões habilitadas, e alterações externas precisariam de IDs estáveis, controle de revisão, idempotência, tombstones de exclusão e regra explícita de conflito. A pessoa poderia usar uma ou ambas as conexões.

Fases escolhidas: agenda local e comandos de IA; feed `.ics` por URL secreta, somente leitura, como primeira integração com iOS; Google via OAuth e CalDAV ficam para uma fase posterior, se houver necessidade de sincronização bidirecional. Se a equipe preferir integração nativa por aplicativo iOS, EventKit também oferece acesso a eventos, mas exige um cliente iOS nativo e permissões do sistema.

Referências oficiais: [adicionar conta CalDAV no iPhone](https://support.apple.com/guide/iphone/iphc37be2016/27/ios), [assinar calendário `.ics` no iPhone](https://support.apple.com/guide/iphone/iph3d1110d4/27/ios), [EventKit](https://developer.apple.com/documentation/eventkit) e [operações de eventos da API Google Calendar](https://developers.google.com/workspace/calendar/api/v3/reference/events).

## Limites iniciais

Não planejar colaboração entre usuários ou convites a terceiros. Sincronização bidirecional com CalDAV/Google é posterior à consistência da agenda local e à definição de conflitos, calendário padrão, privacidade e recuperação de falhas.

O calendário pode oferecer uma camada visual de feriados nacionais e estaduais consultados por ano/UF. A fonte BrasilAPI não cobre todos os feriados municipais; não apresentar a lista como calendário legal completo nem deslocar vencimentos, tarefas ou compromissos automaticamente. Se a consulta estiver indisponível, os eventos pessoais continuam funcionando.
