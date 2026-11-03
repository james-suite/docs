---
id: enum-party-kind
title: Enum — PartyKind
description: Valores e invariantes propostos de PartyKind.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, pessoas-empresas
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Person` | `person` | Pessoa natural |
| `Organization` | `organization` | Empresa ou organização |

## Regras e transições

Uma Party possui um tipo. Converter preserva referências e exige revisão dos campos específicos; não deduzir pelo nome.
