---
id: escopo-2-0
title: Módulos do James
description: Módulos do James preservados e evoluções planejadas com profundidade progressiva.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: financas, tags, veiculos, diario, migrations, enums, decisoes-e-pendencias
source_refs:
updated: 2026-10-26
---

## Núcleo que deve migrar

[Finanças](doc:financas), [cartões](doc:cartoes), [tags](doc:tags), [acertos](doc:acertos), [pessoas/contatos](doc:pessoas-empresas), [notificações](doc:notificacoes) e [auditoria](doc:auditoria-2-0). Preservar funcionalidades e dados úteis do 1.x, registrando mudanças que alterem relatórios ou regras.

Melhorias centrais: vários pagamentos por transação, pagamentos de fatura independentes, arquivamento de instrumentos, classificação criada no formulário, automação de tags com revisão e central de atenção útil.

## Módulos planejados

| Área | Primeira fatia e vínculos |
| --- | --- |
| [Veículos](doc:veiculos) | Cadastro independente de patrimônio, abastecimento preenchido por nota fiscal, custos ligados a itens financeiros, serviço com data real, diário genérico, odômetro e checklist. |
| [Produtos e preços](doc:produtos-precos) | Catálogo pessoal por código/apresentação, preços normalizados vindos de compras e comparação por lugar; cesta “Minha inflação” como evolução posterior. |
| [Estoque e despensa](doc:estoque-despensa) | Itens alimentares e gerais, quantidades e lotes, mapa de locais, validade, contagem e reposição; compras financeiras podem abastecer o estoque, mas não são obrigatórias. |
| [Agenda e tarefas](doc:agenda-tarefas) | Compromissos, checklists, pendências e lembretes; não duplica fatos financeiros. |
| [Diário](doc:diario) | Texto, revisões, anexos e contexto da vida; conecta fatos reais e preserva narrativa original. |
| [Lugares e check-ins](doc:lugares-checkins) | Check-in manual no site, com posição opcional obtida pelo navegador após ação e permissão. |
| [Patrimônio](doc:patrimonio) | Bens, avaliações e documentos sem duplicar ativos financeiros ou veículos. |
| [Saúde e treinos](doc:saude-treinos) | Rotinas, sessões e medições tipadas, privadas e independentes de IA. |
| [IA e memória](doc:ia-e-memoria) | Sugestão de classificação no início; busca/assistente com fontes em etapa posterior. |

## Corte do primeiro lançamento

Essas áreas estão planejadas; todas não precisam ser construídas antes da primeira migração real. O primeiro marco utilizável deve preservar o núcleo atual com dados reconciliados. Veículos, diário e captura podem ser entregues nas próximas fatias sem quebrar o contrato de identidade e portabilidade.

Cada módulo só avança para implementação com campos, estados, ações, autorização, arquivos, auditoria, casos de erro e critérios de aceite definidos. As notas de [migrations](doc:migrations) e [enums](doc:enums) já iniciam esses contratos.

## Evoluções posteriores

Colaboração entre usuários, conta bancária conjunta, integração Apple Health, sincronização bancária, chat com acesso amplo, embeddings e ações autônomas não são pré-requisitos do núcleo 2.0. Precisam de jornadas e decisões próprias, mantendo os dados pessoais isolados.
