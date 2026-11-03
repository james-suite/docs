---
id: enum-classification-mode
title: Enum — ClassificationMode
description: Modos propostos de revisão e aplicação da classificação assistida.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, ia
related: planejamento-2-0, enums, tags, ia-e-memoria, migration-12-ai
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Suggest` | `suggest` | Sugestões aguardam revisão humana; padrão. |
| `AutomaticTags` | `automatic_tags` | Aplicação opcional da classificação completa, incluindo criação e atribuição de tags específicas ausentes, com validação de revisão e política autorizada pelo usuário. |

No modo Suggest, a pessoa aplica ou descarta o conjunto de classificação; tags específicas ausentes são criadas junto com a aplicação, sem confirmação individual. AutomaticTags só pode ser ativado explicitamente e nunca substitui atribuições manuais, aplica resultados antigos ou modifica valores financeiros e outros campos. Categorias financeiras ficam fora da classificação por IA. A preferência guarda sua versão para explicar qual política autorizou a atribuição.

Não usar a confiança autodeclarada do modelo como garantia. O modo automático só avança após avaliação com exemplos representativos e comportamento de desfazer. Falha ou resultado inválido vira resultado pendente/erro controlado, nunca criação arbitrária de tags.

## Implementação

- [ ] Criar enum e cast em ai_user_settings com padrão Suggest.
- [ ] Revalidar autorização, escopo, revisão, especificidade e atribuições manuais antes de aplicar.
- [ ] Auditar como sistema sob política autorizada, sem inventar revisão humana.
- [ ] Desfazer as atribuições daquela execução sem remover mudanças humanas posteriores.
