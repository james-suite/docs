---
id: enum-calendar-command-action
title: Enum — CalendarCommandAction
description: Operações explícitas de agenda que podem ser preparadas por linguagem natural.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, ia-e-memoria, agenda-tarefas
source_refs:
updated: 2026-10-28
---

## Contrato proposto

Esta nota planeja uma enum string para a operação que uma proposta de agenda pretende aplicar. Não habilita execução direta pelo modelo.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Create` | `create` | Criar compromisso na agenda James. |
| `Update` | `update` | Alterar campos de um compromisso existente, com alvo e revisão identificados. |
| `Cancel` | `cancel` | Manter o compromisso no histórico com estado `cancelled`. |
| `Delete` | `delete` | Mover o compromisso para a lixeira, preservando recuperação e auditoria. |

## Regras

`Update`, `Cancel` e `Delete` exigem um alvo único e confirmação após mostrar o evento escolhido. Uma proposta com mais de um possível alvo solicita esclarecimento. `Create` inequívoco pode ser salvo automaticamente somente com preferência explícita habilitada; o resultado deve oferecer desfazer.

## Persistência

Aplicar em `calendar_command_proposals` conforme [migrations](doc:migrations). Revalidar autorização, revisão do evento, fuso, campos e disponibilidade antes de aplicar a operação de domínio.

- [ ] Revisar os cases no fluxo de [agenda](doc:agenda-tarefas) e [IA](doc:ia-e-memoria).
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições e exclusão reversível.
- [ ] Cobrir alvos ambíguos, revisão concorrente e confirmação expirada.
