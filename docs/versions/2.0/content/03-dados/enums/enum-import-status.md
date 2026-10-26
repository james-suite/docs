---
id: enum-import-status
title: Enum — ImportStatus
description: Valores e invariantes propostos de ImportStatus.
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
| `Uploaded` | `uploaded` | Arquivo recebido |
| `Validating` | `validating` | Validando |
| `Ready` | `ready` | Prévia disponível |
| `Running` | `running` | Importando |
| `Completed` | `completed` | Concluída e reconciliada |
| `Failed` | `failed` | Falhou |
| `Cancelled` | `cancelled` | Cancelada |

## Regras e transições

uploaded → validating → ready → running → completed. Cancelar permitido antes da publicação. Falha após lotes gera estado recuperável; completed exige reconciliação, não apenas término do worker.

## Persistência

Aplicar em `import_runs` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:migracao-2-0) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
