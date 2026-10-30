---
id: enum-capture-source
title: Enum — CaptureSource
description: Valores e invariantes propostos de CaptureSource.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, lugares-checkins
source_refs:
updated: 2026-10-29
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Manual` | `manual` | Informado pela pessoa no site |
| `BrowserLocation` | `browser_location` | Posição pontual obtida pelo navegador durante salvamento/check-in iniciado pela pessoa e após permissão explícita |
| `Import` | `import` | Importação pelo site |

## Regras e transições

Guardar método de captura separado da certeza do fato. `BrowserLocation` só é usado quando a pessoa inicia uma ação no site, habilitou captura no salvamento (ou acionou o check-in) e o navegador fornece uma posição. O case se aplica a `visits` e `activity_location_contexts`; não significa que um log prove presença no lugar associado ao registro.

## Persistência

Aplicar em `visits` e `activity_location_contexts` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:lugares-checkins) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
