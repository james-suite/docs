---
id: migration-17-calendar
title: Migration 17 — Agenda e séries
description: Contrato lógico de tabelas, FKs, índices e conversão: agenda, séries e conectores planejados.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, agenda-tarefas, componentes-tecnicos, ia-e-memoria
source_refs: https://datatracker.ietf.org/doc/html/rfc5545, https://support.apple.com/guide/iphone/iphc37be2016/27/ios, https://support.apple.com/guide/iphone/iph3d1110d4/27/ios, https://developers.google.com/workspace/calendar/api/v3/reference/events
updated: 2026-11-07
---

## `calendar_series`

| Campo ou grupo | Contrato |
| --- | --- |
| `title, timezone, recurrence_spec, revision` | Regra versionada e subconjunto de repetição validado. |
| `starts_at ou starts_on` | Instante ou data de dia inteiro, com tipo explícito. |

## `calendar_events`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, series_id, original_occurrence_key` | Identidade e exceção de série opcional. |
| `title, state, starts_at, ends_at, starts_on, ends_on, all_day, timezone, revision` | CalendarEventState; datas de dia inteiro separadas de instantes, com coerência por CHECK. Revisão monotônica permite rejeitar propostas e sincronizações obsoletas. |
| `place_id, notes, cancelled_at` | Lugar e observações; FKs adicionadas após pacote 13. |
| `deleted_at, deleted_by_user_id` | Exclusão reversível com auditoria; diferenciar de `cancelled`, que permanece visível no histórico. |

## Integridade, índices e comportamento

- UNIQUE(space_id, series_id, original_occurrence_key) quando série presente.
- Dia inteiro usa intervalo de datas com início inclusivo e fim exclusivo. Um evento de um dia termina na data seguinte. Eventos com hora preservam timezone mesmo fora de série. Fim explícito deve ser posterior ao início. Duração desconhecida mantém ends_at nulo e é apresentada como tal.
- Editar esta ocorrência e próximas ocorrências são operações diferentes; preservar histórico e lembretes.
- Feriados nacionais/estaduais consultados pela BrasilAPI são uma camada de leitura/cache, não `calendar_events` pessoais; a integração é contextual, não altera vencimentos e não promete feriados municipais.
- A FK calendar_command_proposals.target_event_id entra na extensão opcional quando os pacotes 12 e 17 existirem. Agenda manual é independente da IA.
- Exclusão gera tombstone suficiente para propagar remoção aos conectores e evitar que a sincronização recrie o evento.

## Feed `.ics` por URL secreta — primeira integração de calendário

O James oferece um feed de assinatura somente leitura, acessível por URL secreta. O iPhone busca esse feed periodicamente; mudanças feitas no James aparecem após a próxima atualização do Calendário. Alterações feitas na cópia assinada no iPhone não voltam ao James. O feed contém o conjunto autorizado de eventos e os cancelamentos necessários à atualização dos clientes. Um tombstone expõe somente os metadados mínimos da ocorrência removida. A URL funciona como credencial de leitura.

| Tabela | Campos e invariantes |
| --- | --- |
| `calendar_feed_tokens` | `space_id, owner_user_id, name, token_hash, expires_at, revoked_at, last_accessed_at`. UNIQUE(token_hash); segredo aleatório de alta entropia mostrado uma vez e armazenado somente como hash. Usuário pode revogar/rotacionar cada feed e ver último acesso. |

URL do feed funciona como credencial, não como uma sessão autenticada do James: exigir HTTPS, limitar a dados do dono/espaço, usar respostas privadas sem cache compartilhado, remover o token de logs e referenciadores, permitir expiração e revogação imediata. A emissão do link requer aviso de que qualquer pessoa com a URL poderá ler os eventos até sua revogação.

## Sincronização bidirecional — fase posterior

Uma conta CalDAV do James pode ser adicionada no Calendário do iPhone como conta própria. Google Calendar usa OAuth e a API Google Calendar. Qualquer conexão só fica ativa se a pessoa habilitar.

Antes de habilitar sincronização bidirecional, planejar a extensão destas tabelas:

| Tabela candidata | Campos e invariantes |
| --- | --- |
| `calendar_connections` | `space_id, owner_user_id, provider_code, transport, external_calendar_id, status, credential_reference, selected_as_default, last_synced_at`. Credencial OAuth é criptografada/referenciada; não persistir senha Apple/Google em claro. Uma conta pode ter mais de um calendário. |
| `calendar_event_external_links` | `connection_id, calendar_event_id, external_event_id, external_revision, sync_state, last_synced_at`. UNIQUE por evento/conexão e por ID externo/conexão; identidade externa nunca é inferida só pelo título. |
| `calendar_sync_runs` | `connection_id, direction, idempotency_key, cursor, status, started_at, completed_at, error_code`. Cursors/sync tokens pertencem à conexão e precisam ser atualizados atomicamente com os eventos aceitos. |

As tabelas de conector só entram na migration física quando CalDAV/Google tiverem protótipo e regras aprovadas. Resolver edição concorrente com revisão/ETag ou equivalente e escolha explícita; não sobrescrever silenciosamente eventos alterados fora do James. Respeitar exclusão, escopo por usuário e revogação do conector. O James não terá app iOS nem integração com EventKit ou Lembretes Apple.

## Identidade de calendário e ocorrências

Persistir ics_uid estável, sequência de revisão e chave nominal da ocorrência. DTSTART inicia o intervalo e DTEND expressa o fim exclusivo. UID identifica o componente entre atualizações e SEQUENCE informa sua revisão. Séries usam identidade de ocorrência original para que mover uma ocorrência preserve sua referência. Gerar texto com escape e serializador compatível com iCalendar. [RFC 5545](https://datatracker.ietf.org/doc/html/rfc5545).

O protótipo deve validar atualização, cancelamento e exclusão no Calendário do iPhone. Planejar tombstones com UID, revisão e sinalização de cancelamento, prazo e janela de publicação definidos. A remoção física depende desse prazo e da política local. O feed oferece conteúdo mínimo configurável, e anexos privados permanecem acessíveis pela autorização do James.

Validar token e revogação antes de processar ETag ou responder 304. Separar cache por feed/revisão. Rotação apresenta um novo segredo e a interface orienta atualizar a assinatura.

Repetição usa horário local e fuso da série. Datas/horas ambíguas ou inexistentes por mudança de fuso exigem política visível e confirmação quando necessário. Materializar uma janela limitada com chaves únicas e preservar exceções e cancelamentos ao repetir o job. Dividir uma série conserva o histórico anterior e explicita a identidade das próximas ocorrências.

## Conversão do James 1.x

Módulo novo. Datas financeiras aparecem como projeção/ligação; não copiar cada vencimento para um evento mutável independente.

## Componentes previstos

SaveCalendarEvent; UpdateCalendarEvent; CancelCalendarEvent; DeleteCalendarEvent; SplitCalendarSeries; CancelOccurrence; ReminderPlanner. Para fase de integração: IcsFeed, CalDavEndpoint, GoogleCalendarAdapter, CalendarSyncPlanner e CalendarConflictResolver. Os nomes são candidatos; não criam compromisso de implementação.
