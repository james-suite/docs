---
id: home
title: James
description: Planejamento de produto e desenvolvimento dos módulos do James.
type: overview
status: proposed
visibility: public
tags: james/2.0, planejamento
related: planejamento-2-0, modelo-de-dados, migrations, enums, fases-2-0, migracao-2-0
source_refs:
updated: 2026-10-26
---

## Comece pelo plano

O planejamento preserva a experiência dos módulos atuais e os conecta melhor: finanças com vários pagamentos, cartões, tags assistidas, empresas, veículos, diário e histórico de lugares.

- [Planejamento completo](doc:planejamento-2-0), [escopo](doc:escopo-2-0) e [decisões](doc:decisoes-e-pendencias).
- [Modelo de dados](doc:modelo-de-dados), [migrations](doc:migrations), [enums](doc:enums) e [componentes técnicos](doc:componentes-tecnicos).
- [Migração por arquivo no navegador](doc:migracao-2-0) e [compatibilidade com o James atual](doc:compatibilidade-legado).
- [Fases de desenvolvimento](doc:fases-2-0).

O desenvolvimento dos módulos planejados começa em 2027, com o starter kit disponível naquele momento e Flux 2 gratuito. O plano ainda contém propostas a revisar; nenhum schema planejado foi aplicado à versão atual do James.

## Mapa do planejamento

O conteúdo está dividido por assunto, tanto nas fontes quanto na navegação lateral:

| Pasta | O que fica nela |
| --- | --- |
| `00-planejamento/` | Visão geral, escopo, fases e decisões abertas. |
| `01-arquitetura/` | Arquitetura, modelo de dados, UI/UX, auditoria e componentes técnicos. |
| `02-dominios/financas/` | Finanças, cartões e acertos. |
| `02-dominios/organizacao/` | Tags, pessoas/empresas, notificações, agenda e tarefas. |
| `02-dominios/memoria-e-contexto/` | Diário, lugares/check-ins e IA/memória. |
| `02-dominios/bens-e-saude/` | Veículos, patrimônio e saúde/treinos. |
| `03-dados/` | Catálogos e especificações detalhadas de enums e migrations. |
| `04-migracao/` | Importação do arquivo e compatibilidade com o James 1.x. |

### Módulos financeiros

[Finanças](doc:financas) · [Cartões](doc:cartoes) · [Acertos](doc:acertos).

### Organização pessoal

[Tags](doc:tags) · [Pessoas/empresas](doc:pessoas-empresas) · [Notificações](doc:notificacoes) · [Agenda/tarefas](doc:agenda-tarefas).

### Memória e contexto

[Diário](doc:diario) · [Lugares/check-ins](doc:lugares-checkins) · [IA e memória](doc:ia-e-memoria).

### Bens e saúde

[Veículos](doc:veiculos) · [Patrimônio](doc:patrimonio) · [Saúde/treinos](doc:saude-treinos).

[Consulte a documentação da versão 1.x](versions/1.x/), com as funcionalidades publicadas atualmente.
