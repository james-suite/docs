---
id: migration-11-notifications
title: Migration 11 — Notificações e outbox
description: Contrato lógico de tabelas, FKs, índices e conversão: notificações e outbox.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, notificacoes, componentes-tecnicos
source_refs:
updated: 2026-10-25
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `notifications`

| Campo ou grupo | Contrato |
| --- | --- |
| `Estrutura nativa + space_id` | UUID e destinatário nativo; escopo pessoal persistido. |
| `type_code, payload_version, source_record_id` | Tipo permitido, versão e origem. |
| `level, payload, occurrence_key` | Mensagem estruturada e deduplicação durável. |
| `read_at, dismissed_at, snoozed_until` | Estados pessoais de leitura, arquivamento e adiamento. |

## `notification_preferences`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id, type_code, channel` | Chave única por usuário/tipo/canal. |
| `enabled, delivery_mode, quiet_hours, timezone` | Preferências validadas, frequência e horário. |

## `notification_endpoints`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id, channel, encrypted_configuration` | Destino verificado e segredo protegido quando houver. |
| `verified_at, revoked_at` | Verificação e revogação. |

## `notification_deliveries`

| Campo ou grupo | Contrato |
| --- | --- |
| `notification_id, channel, endpoint_id, delivery_key` | Intenção/outbox com chave única por ocorrência e canal. |
| `message_snapshot, scheduled_at, claimed_at, finalized_at` | Snapshot seguro e estado de recuperação do processamento. |

## `notification_delivery_attempts`

| Campo ou grupo | Contrato |
| --- | --- |
| `delivery_id, attempt_number, status` | UNIQUE(delivery_id, attempt_number), DeliveryStatus. |
| `provider_reference, started_at, finished_at, failure_code` | Diagnóstico sanitizado; sem segredo ou exceção bruta. |

## Integridade, índices e comportamento

- Persistir intenção na transação de domínio e enviar após commit. Worker e varredura da outbox retomam intenções órfãs; afterCommit sozinho não garante entrega.
- Uma tentativa nova não cria outra notificação. UNIQUE por destinatário/tipo/origem/ocorrência garante deduplicação.
- Leitura interna é diferente de aceitação externa. Mensagem antiga não concede acesso ao alvo atual.
- Timeout e retry_after compatíveis, backoff e limite de tentativas. Eventos resolvidos cancelam lembretes ainda não enviados.

## Conversão do James 1.x

Importar leitura e payload legados compatíveis como histórico. Não criar deliveries novas nem reenviar Telegram/e-mail ao importar. Preferências globais antigas são apresentadas para configuração pessoal, sem transportar tokens.

## Componentes previstos

CreateDomainNotification; RequestDelivery; SendDelivery; RetryDelivery; ResolveReminder; digest por timezone.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
