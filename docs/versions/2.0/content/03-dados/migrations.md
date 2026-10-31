---
id: migrations
title: Migrations planejadas
description: Ordem lógica de criação das tabelas, dependências e contratos por domínio.
type: development-hub
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, modelo-de-dados, enums, componentes-tecnicos, fases-2-0, compatibilidade-legado
source_refs:
updated: 2026-10-31
---

## Como usar este catálogo

Cada nota abaixo define campos, FKs, índices, invariantes, componentes e conversão do legado. Ela pode originar várias migrations físicas, uma por responsabilidade. Nenhum schema novo foi aplicado ao James atual. Na implementação de 2027, o baseline nativo vem do starter kit escolhido.

Números são identificadores editoriais. Dependências adicionais devem ser resolvidas com migration de FK posterior, sem desativar checagens do banco. Começar por identidade/escopo, depois catálogos, documentos, pagamentos, cartões e conversão. A camada ledger é uma proposta sujeita ao protótipo; não iniciar sua implementação como decisão já aprovada.

## Ordem lógica

- [Migration 00 — Base técnica do starter kit](doc:migration-00-base) — fundação técnica.
- [Migration 01 — Identidade e espaço pessoal](doc:migration-01-spaces) — fundação técnica.
- [Migration 02 — Registros vinculáveis e proveniência](doc:migration-02-records) — estrutura transversal.
- [Migration 03 — Pessoas, empresas e grupos](doc:migration-03-parties) — pessoas e empresas.
- [Migration 04 — Tags, categorias e regras locais](doc:migration-04-tags) — classificação.
- [Migration 05 — Contas financeiras e contrapartidas](doc:migration-05-accounts) — finanças.
- [Migration 06 — Transações, itens e origem fiscal](doc:migration-06-transactions) — finanças.
- [Migration 07 — Pagamentos, alocações e saldo](doc:migration-07-payments) — finanças.
- [Migration 08 — Cartões, ciclos e parcelas](doc:migration-08-cards) — cartões.
- [Migration 09 — Recorrências e ocorrências](doc:migration-09-recurrences) — finanças.
- [Migration 10 — Rateios e liquidações de acertos](doc:migration-10-settlements) — acertos.
- [Migration 11 — Notificações e outbox](doc:migration-11-notifications) — notificações.
- [Migration 12 — IA, credenciais e sugestões](doc:migration-12-ai) — IA.
- [Migration 13 — Lugares e check-ins](doc:migration-13-places) — lugares e check-ins.
- [Migration 14 — Diário e revisões](doc:migration-14-journal) — diário.
- [Migration 15 — Veículos, serviços e combustível](doc:migration-15-vehicles) — veículos.
- [Migration 16 — Tarefas e checklists](doc:migration-16-tasks) — agenda e tarefas.
- [Migration 17 — Agenda e séries](doc:migration-17-calendar) — agenda e tarefas.
- [Migration 18 — Bens e avaliações](doc:migration-18-assets) — patrimônio.
- [Migration 19 — Saúde e treinos](doc:migration-19-health) — saúde e treinos.
- [Migration 20 — Exportação, importação e mapeamento legado](doc:migration-20-imports) — portabilidade.
- [Migration 21 — Catálogo de produtos e preços observados](doc:migration-21-products) — catálogo pessoal, preços comparáveis e cestas.

A extensão opcional futura do pacote 12 para conversas, memória derivada e busca vetorial está descrita em [Migration 12 — IA](doc:migration-12-ai) e no [plano do assistente pessoal](doc:assistente-pessoal-rag); ela não adiciona tabelas obrigatórias à sequência acima.

A conexão opcional com Apple Health estende a [Migration 19 — Saúde e treinos](doc:migration-19-health), sem novo pacote de domínio. O plano propõe sincronização iniciada pela pessoa e importação XML manual; nenhuma dessas opções é requisito da primeira migração do James.

## Evolução sem tabelas prematuras

A sequência não obriga construir módulos futuros antes de entregar a base financeira migrável. A preferência e o contexto geográfico por ação são previstos na base/identidade; lugares e visitas/check-ins têm seu contrato de domínio no pacote 13. Veículos, agenda, patrimônio e saúde recebem migrations quando seus marcos começarem. Conversas, embeddings e memória derivada do assistente do James ficam como extensão futura opcional, sem tabelas ou extensão pgvector obrigatórias no primeiro lançamento.

Referências cruzadas: [Enums](doc:enums), [componentes técnicos](doc:componentes-tecnicos) e [compatibilidade com o 1.x](doc:compatibilidade-legado).
