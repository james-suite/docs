---
id: migration-11-notifications
title: Migration 11 — Notificações e outbox
description: Contrato lógico de tabelas, FKs, índices e conversão: notificações e outbox.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, notificacoes, componentes-tecnicos, enum-delivery-status
source_refs: https://developer.apple.com/documentation/usernotifications/sending-web-push-notifications-in-web-apps-and-browsers, https://www.rfc-editor.org/rfc/rfc8030, https://www.rfc-editor.org/rfc/rfc8291, https://www.rfc-editor.org/rfc/rfc8292
updated: 2026-11-01
---

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
| `id, user_id, channel, encrypted_configuration` | Um destino por endpoint; Web Push guarda a assinatura e suas chaves cifradas. Permitir vários endpoints Web Push por usuário. |
| `subscription_fingerprint, device_label, last_seen_at, expires_at` | Fingerprint com UNIQUE por usuário/canal para impedir a mesma assinatura duplicada, nome opcional do dispositivo e vigência; não registrar a URL em logs. |
| `verified_at, revoked_at` | Verificação e revogação pelo usuário ou pelo sistema após endpoint inválido. |

## `notification_deliveries`

| Campo ou grupo | Contrato |
| --- | --- |
| `notification_id, channel, endpoint_id, delivery_key` | Uma intenção por endpoint; chave idempotente inclui notificação, canal e endpoint. Push pode gerar várias entregas para dispositivos diferentes sem duplicar a notificação central. |
| `message_snapshot, scheduled_at, claimed_at, finalized_at` | Snapshot seguro e estado de recuperação do processamento. |

## `notification_delivery_attempts`

| Campo ou grupo | Contrato |
| --- | --- |
| `delivery_id, attempt_number, status` | UNIQUE(delivery_id, attempt_number), DeliveryStatus. |
| `provider_reference, started_at, finished_at, failure_code` | Diagnóstico sanitizado; sem segredo ou exceção bruta. |

## Integridade, índices e comportamento

- Não persistir um segundo estado na outbox: derivar pendência, processamento, retry e término de `scheduled_at`, lease/`claimed_at`, `finalized_at` e da última tentativa. Ao resolver ou dispensar uma entrega ainda não enviada, registrar uma tentativa `Skipped`; tentativas guardam o histórico de transporte.
- Persistir intenção na transação de domínio e enviar após commit. Worker e varredura da outbox retomam intenções órfãs; afterCommit sozinho não garante entrega.
- Uma tentativa nova não cria outra notificação. UNIQUE por destinatário/tipo/origem/ocorrência garante deduplicação.
- Leitura interna é diferente de aceitação externa. Mensagem antiga não concede acesso ao alvo atual.
- Criar assinatura Web Push apenas depois de a pessoa tocar em “Ativar notificações” e aceitar a permissão. No iOS, exigir web app instalado na Tela de Início; não tratar favorito aberto no Safari como instalação. No Android, a instalação PWA é opcional; verificar suporte às APIs no navegador.
- Uma conta pode ter várias assinaturas Web Push. Revogação no James remove o endpoint; erros permanentes do serviço, como assinatura expirada/inválida, desativam somente aquele endpoint e ficam registrados na tentativa.
- Cifrar endpoint e chaves de assinatura em repouso; manter a chave privada VAPID em segredo de instalação. Push apresenta conteúdo imediatamente e leva só título/contexto mínimo e o identificador da notificação. Todo clique abre a tela `show` dessa notificação; nunca enviar o link direto ao registro de origem no push. A tela consulta o aviso e oferece a ação relacionada após autenticar e autorizar novamente a pessoa e o alvo.
- Timeout e retry_after compatíveis, backoff e limite de tentativas. Eventos resolvidos cancelam lembretes ainda não enviados.

## Conversão do James 1.x

Importar leitura e payload legados compatíveis como histórico. Não criar deliveries nem enviar notificações externas durante a importação. Preferências globais antigas são apresentadas para configuração pessoal, sem transportar tokens ou endpoints.

## Componentes previstos

CreateDomainNotification; RequestDelivery; SendDelivery; RetryDelivery; ResolveReminder; digest por timezone.
