---
id: enum-tag-assignment-origin
title: Enum — TagAssignmentOrigin
description: Valores e invariantes propostos de TagAssignmentOrigin.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, tags
source_refs:
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Manual` | `manual` | Escolha humana |
| `Rule` | `rule` | Regra determinística |
| `Ai` | `ai` | IA aceita por revisão ou política automática explicitamente autorizada |
| `Import` | `import` | Informação importada |

## Regras e transições

A origem acompanha a atribuição. Uma escolha manual não é substituída por automação. Trocar a origem exige operação explícita.

## Persistência

Aplicar em `tag_assignments` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:tags) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
