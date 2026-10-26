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
updated: 2026-10-25
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Suggest` | `suggest` | Sugestões aguardam revisão humana; padrão. |
| `AutomaticExistingTags` | `automatic_existing_tags` | Aplicação opcional de tags existentes, com validação de IDs/revisão e política autorizada pelo usuário. |

Categorias e nomes novos continuam exigindo revisão. Modo automático não substitui tags manuais, não aplica resultados antigos e não modifica dinheiro ou outros campos. A preferência guarda sua versão para explicar qual política autorizou a atribuição.

Não usar a confiança autodeclarada do modelo como garantia. O modo automático só avança após avaliação com exemplos representativos e comportamento de desfazer. Falha ou resultado inválido vira sugestão/erro controlado, nunca criação arbitrária de tags.

## Implementação

- [ ] Criar enum e cast em ai_user_settings com padrão Suggest.
- [ ] Revalidar autorização, escopo, revisão e tags manuais antes de aplicar.
- [ ] Auditar como sistema sob política autorizada, sem inventar revisão humana.
- [ ] Desfazer as atribuições daquela execução sem remover mudanças humanas posteriores.
