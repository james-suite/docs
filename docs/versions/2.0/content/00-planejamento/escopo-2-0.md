---
id: escopo-2-0
title: Módulos do James
description: Módulos do James preservados e evoluções planejadas com profundidade progressiva.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: financas, tags, veiculos, diario, migrations, enums, decisoes-e-pendencias, assistente-pessoal-rag, modulos-e-integracoes, saude-treinos
source_refs:
updated: 2026-10-31
---

## Núcleo que deve migrar

[Finanças](doc:financas), [cartões](doc:cartoes), [tags](doc:tags), [acertos](doc:acertos), [pessoas/contatos](doc:pessoas-empresas), [notificações](doc:notificacoes) e [auditoria](doc:auditoria-2-0). Preservar funcionalidades e dados úteis do 1.x, registrando mudanças que alterem relatórios ou regras.

Melhorias centrais: vários pagamentos por transação, pagamentos de fatura independentes, arquivamento de instrumentos, classificação criada no formulário, automação de tags com revisão e central de atenção útil.

## Módulos planejados

| Área | Primeira fatia e vínculos |
| --- | --- |
| [Veículos](doc:veiculos) | Cadastro independente de patrimônio, abastecimento preenchido por nota fiscal, custos ligados a itens financeiros, serviço com data real, diário genérico, odômetro e checklist. |
| [Produtos e preços](doc:produtos-precos) | Catálogo pessoal por código/apresentação, preços normalizados vindos de compras e comparação por lugar; cesta “Minha inflação” como evolução posterior. |
| [Agenda e tarefas](doc:agenda-tarefas) | Compromissos, checklists, pendências e lembretes; não duplica fatos financeiros. Comandos por linguagem natural e conectores Apple/Google estão propostos, em fases distintas. |
| [Diário](doc:diario) | Texto, revisões, anexos e contexto da vida; conecta fatos reais e preserva narrativa original. |
| [Lugares e check-ins](doc:lugares-checkins) | Check-in manual no site, com posição opcional obtida pelo navegador após ação e permissão. |
| [Patrimônio](doc:patrimonio) | Bens, avaliações e documentos sem duplicar ativos financeiros ou veículos. |
| [Saúde e treinos](doc:saude-treinos) | Rotinas, sessões e medições tipadas e privadas; HealthKit é uma integração opcional posterior, sem depender do primeiro lançamento migrável. |
| [IA](doc:ia-e-memoria) | Primeiro: sugestões de tags para transações/itens e fallback fiscal; comandos de agenda e rascunhos por texto sem nota estão em avaliação. Evolução posterior do assistente pessoal está planejada em [Memória e RAG](doc:assistente-pessoal-rag). |

Consultas opcionais da BrasilAPI complementam os formulários de empresa, endereço, instituição financeira, item fiscal, veículo e agenda. São pesquisas pontuais com resultado revisável, nunca requisito para o James funcionar; usos e limites estão nos [componentes técnicos](doc:componentes-tecnicos).

As conexões entre os módulos existentes e as ideias de novas áreas — documentos e prazos, manutenção da casa, viagens e projetos pessoais — estão em [Módulos candidatos e integrações](doc:modulos-e-integracoes). São propostas para descoberta posterior e não aumentam o escopo do primeiro lançamento.

## Corte do primeiro lançamento

Essas áreas estão planejadas; todas não precisam ser construídas antes da primeira migração real. O primeiro marco utilizável deve preservar o núcleo atual com dados reconciliados. Veículos, diário e captura podem ser entregues nas próximas fatias sem quebrar o contrato de identidade e portabilidade.

Cada módulo só avança para implementação com campos, estados, ações, autorização, arquivos, auditoria, casos de erro e critérios de aceite definidos. As notas de [migrations](doc:migrations) e [enums](doc:enums) já iniciam esses contratos.

## Evoluções posteriores

Colaboração entre usuários, conta bancária conjunta, integração Apple Health e sincronização bancária não são pré-requisitos do núcleo migrável. A integração Apple Health é uma fase opcional do módulo [Saúde e treinos](doc:saude-treinos), com detalhes de HealthKit, cliente iOS, importação XML e privacidade na [Migration 19](doc:migration-19-health). O assistente pessoal do James, com RAG e ferramentas entre módulos, está planejado em fases posteriores e não é pré-requisito do primeiro lançamento. Voz e ações proativas/autônomas continuam decisões separadas; o caminho inicial é busca fundamentada somente leitura, com dados pessoais isolados.
