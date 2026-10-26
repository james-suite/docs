---
id: migration-13-places
title: Migration 13 — Lugares e check-ins
description: Contrato lógico de tabelas, FKs, índices e conversão: lugares e check-ins.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, lugares-checkins, componentes-tecnicos
source_refs:
updated: 2026-10-25
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 03. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `places`

| Campo ou grupo | Contrato |
| --- | --- |
| `name, party_id, address` | Lugar pessoal, empresa/filial opcional e endereço estruturado. |
| `latitude, longitude, timezone` | Coordenadas opcionais com limites; fuso IANA. |

## `visits`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, place_id, started_at, ended_at` | Visita; fim opcional não anterior ao início. |
| `observed_timezone, precision_meters, source, received_at` | Qualidade, CaptureSource e hora de ingestão. |
| `device_id, external_event_id, notes` | Deduplicação de origem e observação privada. |

## `capture_devices`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id, label, token_hash, revoked_at` | Token nunca recuperável, dispositivo e revogação. |
| `scopes, last_seen_at` | Permissões restritas a check-ins e diagnóstico mínimo. |

## Integridade, índices e comportamento

- CHECK latitude -90..90 e longitude -180..180, ambas presentes ou ambas nulas. Precisão não negativa.
- UNIQUE(space_id, device_id, external_event_id) para eventos de integração; dispositivos e lugares do mesmo espaço.
- Presença inferida de compra não entra como visita confirmada. Dados brutos contínuos de GPS ficam para uma migration futura com retenção/volume definidos.

## Conversão do James 1.x

Não há visitas estruturadas para migrar. Notas antigas podem ser vinculadas manualmente; não geocodificar toda a vida financeira sem consentimento.

## Componentes previstos

CreateVisit; CaptureVisitEndpoint; RevokeDevice; PlacesSearch; integração Atalhos iOS como piloto.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
