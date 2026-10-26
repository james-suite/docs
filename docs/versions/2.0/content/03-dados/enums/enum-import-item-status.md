---
id: enum-import-item-status
title: Enum — ImportItemStatus
description: Valores e invariantes propostos de ImportItemStatus.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, migracao-2-0
source_refs:
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Pending` | `pending` | Não aplicado |
| `Imported` | `imported` | Convertido |
| `Skipped` | `skipped` | Ignorado com motivo |
| `Failed` | `failed` | Falha |

## Regras e transições

Um item só pode ser ignorado por regra explícita apresentada na prévia. completed do pacote exige ausência de falha bloqueante; itens importados não são reaplicados em retomada.

## Persistência

Aplicar em `import_items` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:migracao-2-0) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
