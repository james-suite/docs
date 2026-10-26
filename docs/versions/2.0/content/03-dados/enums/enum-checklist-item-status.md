---
id: enum-checklist-item-status
title: Enum — ChecklistItemStatus
description: Valores e invariantes propostos de ChecklistItemStatus.
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
| `Pending` | `pending` | Pendente |
| `Done` | `done` | Conferido |
| `NotApplicable` | `not_applicable` | Não aplicável |

## Regras e transições

Não aplicável exige justificativa quando o modelo requer. Reabrir preserva eventos anteriores; alterar template não altera instância histórica.

## Persistência

Aplicar em `checklist_items` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:veiculos) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
