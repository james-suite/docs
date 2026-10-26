---
id: enum-vehicle-event-type
title: Enum — VehicleEventType
description: Valores e invariantes propostos de VehicleEventType.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, veiculos
source_refs:
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Fueling` | `fueling` | Abastecimento; os dados de volume ficam na tabela específica de combustível |
| `Service` | `service` | Manutenção |
| `Repair` | `repair` | Reparo |
| `Inspection` | `inspection` | Inspeção |
| `Incident` | `incident` | Incidente |
| `Note` | `note` | Diário genérico; não implica despesa ou manutenção concluída |

## Regras e transições

Um evento pode ter vários itens realizados. `Fueling` tem dados próprios. `Note` registra qualquer acontecimento ou observação livre, sem efeito financeiro automático; despesas e pagamentos continuam no módulo financeiro. Task concluída não é automaticamente evento de serviço.

## Persistência

Aplicar em `vehicle_events` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:veiculos) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
