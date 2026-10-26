---
id: tags
title: Tags, categorias e criação no combobox
description: Classificação financeira, etiquetas transversais e sugestões assistidas por IA.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, financas, ia-e-memoria, componentes-tecnicos, enums, migrations
source_refs: https://github.com/sge-suite/sge/blob/master/resources/views/components/select.blade.php, https://github.com/sge-suite/sge/blob/master/resources/views/components/select-create.blade.php, https://github.com/james-suite/james/blob/master/app/Models/FinancialTag.php
updated: 2026-10-25
---

## Melhorar sem perder as tags atuais

Propor dois conceitos complementares: uma categoria financeira por item para relatórios sem dupla contagem e várias tags livres para contexto. “Alimentação” pode ser a categoria e “viagem”, “trabalho” e “reembolsável” podem ser tags. Essa separação precisa de revisão do usuário antes de congelar a UX; a importação manterá a informação original de `is_primary` para permitir corrigir o mapeamento.

Tags deixam de ser exclusivamente financeiras: também podem classificar diário, veículos, lugares, tarefas e documentos. Cada pessoa tem seu catálogo. Categorias podem ter hierarquia curta; suas relações precisam impedir ciclos. Tags têm nome normalizado, descrição, cor, ícone, aliases, arquivamento e fusão com histórico.

## Combobox com criação no formulário

O SGE possui `select.blade.php` com semântica de combobox e o wrapper `select-create.blade.php`, que ativa busca e criação. O atual componente aceita uma seleção ou texto novo. Para o James, planejar um `TagPicker` com várias seleções, chips removíveis e uma ação explícita “Criar tag …”, usando Flux 2 gratuito e o componente próprio como referência.

- Buscar somente no catálogo da pessoa autenticada e mostrar também aliases.
- Valores existentes trafegam por ID; textos novos ficam como seleções pendentes até o salvamento.
- Ao salvar o formulário, resolver/criar as tags na mesma transação. Cancelar o formulário não deve deixar dezenas de tags órfãs.
- Uma restrição única por espaço e nome normalizado resolve criação concorrente. Se outra requisição criou o mesmo nome, reutilizar a tag existente.
- Não criar duplicatas apenas por caixa, espaços ou acento. Mostrar o nome encontrado antes de oferecer uma nova tag.
- Suportar teclado, foco, busca sem resultados, erros por campo e várias seleções. Não depender de componentes Flux Pro.

## Aplicação e herança

Tags do cabeçalho classificam a transação completa. Tags do item são específicas. A busca pode calcular a união, mas não deve persistir cópias invisíveis em todos os itens. Relatórios por categoria usam a classificação do item e um fallback de cabeçalho explicitamente definido. Uma tag usada em vários itens não multiplica o valor do gasto.

A atribuição guarda origem (`manual`, `rule`, `ai`, `import`), responsável e, quando existir, execução de IA. Escolha manual tem prioridade e não é substituída por sugestão posterior.

## Automação inicial

Priorizar regras locais explicáveis por empresa, texto, histórico e tipo de operação. A IA entra como sugestão de tags/categoria quando essas regras não bastarem, com retorno estruturado limitado aos IDs autorizados. Nova tag sugerida pela IA exige confirmação; não criar um catálogo interminável automaticamente.

Sugestões pendentes mostram motivo resumido, origem e opções aceitar/recusar. Uma opção pessoal pode automatizar somente tags existentes após validação do modo, com desfazer e sem sobrepor escolhas humanas; categorias e novos nomes continuam sujeitos à revisão. Não tratar a “confiança” declarada pelo modelo como probabilidade calibrada. Se o registro mudou depois da solicitação, a sugestão fica desatualizada e não é aplicada. Correções podem alimentar regras do próprio usuário sem treinar um modelo nem compartilhar dados entre pessoas.

## Migração

Preservar todas as tags e seus vínculos. A tag primária pode gerar uma categoria com o mesmo nome; a mesma tag pode continuar disponível como etiqueta. Em conflitos entre cabeçalho e item, preservar a primária do item e relatar o fallback utilizado.

As tags especiais atuais usam IDs fixos para reembolso, juros, saldo inicial, transferência, ajuste e pagamento parcial. No 2.0 essas funções pertencem ao tipo da operação e aos lançamentos financeiros. Preservar os rótulos legados; não continuar tomando decisões de saldo com base no número da tag.
