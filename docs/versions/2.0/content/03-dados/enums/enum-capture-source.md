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
updated: 2026-10-26
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Manual` | `manual` | Informado pela pessoa no site |
| `BrowserLocation` | `browser_location` | Posição obtida pelo navegador após ação e permissão da pessoa |
| `Import` | `import` | Importação pelo site |

## Regras e transições

Guardar método de captura separado da certeza do fato. `BrowserLocation` só é usado quando a pessoa aciona a captura de posição no formulário do site e o navegador a fornece.

## Persistência

Aplicar em `visits` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:lugares-checkins) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
