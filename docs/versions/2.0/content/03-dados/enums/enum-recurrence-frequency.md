---
id: enum-recurrence-frequency
title: Enum — RecurrenceFrequency
description: Valores e invariantes propostos de RecurrenceFrequency.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas
source_refs:
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Daily` | `daily` | Diária |
| `Weekly` | `weekly` | Semanal |
| `Monthly` | `monthly` | Mensal |
| `Yearly` | `yearly` | Anual |

## Regras e transições

Intervalo inteiro positivo complementa a frequência. Preservar âncora original em dias inexistentes no mês; não acumular deriva de data.

## Persistência

Aplicar em `financial_recurrences` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:financas) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
