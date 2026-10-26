---
id: enum-ai-task
title: Enum — AiTask
description: Valores e invariantes propostos de AiTask.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, ia-e-memoria
source_refs:
updated: 2026-10-26
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `TagSuggestion` | `tag_suggestion` | Sugerir tags |
| `CategorySuggestion` | `category_suggestion` | Sugerir categoria |
| `FiscalExtraction` | `fiscal_extraction` | Extrair candidatos estruturados de documento fiscal quando um leitor determinístico não atender e o usuário permitir o envio |

## Regras e transições

Conjunto inicial restrito. Resumo de diário, chat, extração de fatos pessoais e ações autônomas precisam de contrato próprio antes de novos cases. FiscalExtraction só processa a nota vinculada à execução, sob permissão e orçamento próprios; não confirma compra, pagador ou pagamento.

## Persistência

Aplicar em `ai_runs` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:ia-e-memoria) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
