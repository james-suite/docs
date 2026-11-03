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
updated: 2026-11-03
---

## Como usar este catálogo

As notas são contratos de planejamento, não migrations PHP executáveis nem schema já aplicado. Cada pacote pode originar várias migrations físicas, separadas por responsabilidade; DDL e conversão de dados ficam em etapas distintas. Na implementação de 2027, o baseline nativo vem do starter kit escolhido. Os campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados); cada página detalha tabelas, invariantes e conversão próprias.

Os números são identificadores editoriais, não timestamps nem ordem obrigatória de arquivos. A matriz registra dependências lógicas; quando uma FK depende de tabela criada depois, adicioná-la em uma migration posterior e manter as checagens do banco ativas. O ledger continua condicionado ao protótipo descrito em [Arquitetura](doc:arquitetura-2-0).

## Pacotes e dependências

| Pacote | Área | Dependências lógicas |
| --- | --- | --- |
| [00 — Base técnica do starter kit](doc:migration-00-base) | Fundação | Starter kit e pacotes escolhidos em 2027 |
| [01 — Identidade e espaço pessoal](doc:migration-01-spaces) | Fundação | 00 |
| [02 — Registros vinculáveis e proveniência](doc:migration-02-records) | Base transversal | 01 |
| [03 — Pessoas, empresas e grupos](doc:migration-03-parties) | Pessoas e empresas | 02 |
| [04 — Tags, categorias e regras locais](doc:migration-04-tags) | Classificação | 02, 03 |
| [05 — Contas financeiras e contrapartidas](doc:migration-05-accounts) | Finanças | 01, 03 |
| [06 — Transações, itens e origem fiscal](doc:migration-06-transactions) | Finanças | 02, 03, 04, 22 |
| [07 — Pagamentos, alocações e saldo](doc:migration-07-payments) | Finanças | 05, 06 |
| [08 — Cartões, ciclos e parcelas](doc:migration-08-cards) | Cartões | 05, 07 |
| [09 — Recorrências e ocorrências](doc:migration-09-recurrences) | Finanças | 04, 06, 08 |
| [10 — Rateios e liquidações de acertos](doc:migration-10-settlements) | Acertos | 03, 06, 07, 22 |
| [11 — Notificações e outbox](doc:migration-11-notifications) | Notificações | 02 |
| [12 — IA, credenciais e sugestões](doc:migration-12-ai) | IA | 02, 04, 06 |
| [13 — Lugares e check-ins](doc:migration-13-places) | Lugares | 02, 03 |
| [14 — Diário e revisões](doc:migration-14-journal) | Diário | 02, 04, 22 |
| [15 — Veículos, serviços e combustível](doc:migration-15-vehicles) | Veículos | 02, 03, 06, 13, 22 |
| [16 — Tarefas e checklists](doc:migration-16-tasks) | Organização | 02, 15 |
| [17 — Agenda e séries](doc:migration-17-calendar) | Organização | 02, 12, 13, 16 |
| [18 — Bens e avaliações](doc:migration-18-assets) | Patrimônio | 02, 03, 06, 15, 22 |
| [19 — Saúde e treinos](doc:migration-19-health) | Saúde | 02, 16 |
| [20 — Exportação, importação e mapeamento legado](doc:migration-20-imports) | Portabilidade | 00–11, 22 |
| [21 — Catálogo de produtos e preços observados](doc:migration-21-products) | Produtos e preços | 02, 03, 06, 13 |
| [22 — Documentos, versões e anexos compartilhados](doc:migration-22-documents) | Documentos e anexos | 00, 02, 03 |

## Checklist comum de implementação

- [ ] Confirmar o contrato com o fluxo do domínio e revisar escopo, cardinalidade e invariantes.
- [ ] Gerar migrations físicas separadas para tabelas, FKs, índices e constraints; verificar instalação vazia e ordem real.
- [ ] Criar Models, casts, factories e seeds somente quando necessários ao pacote.
- [ ] Implementar validação e autorização no limite da operação, respeitando o mesmo espaço.
- [ ] Cobrir concorrência, conversão, reconciliação e falhas específicas do pacote.
- [ ] Incluir auditoria e portabilidade quando os dados forem relevantes para histórico ou exportação.

A extensão opcional futura do pacote 12 para conversas, memória derivada e busca vetorial está descrita em [Migration 12 — IA](doc:migration-12-ai) e no [plano do assistente pessoal](doc:assistente-pessoal-rag); ela não adiciona tabelas obrigatórias à sequência acima.

Uma importação manual do Apple Saúde, se aprovada após validar um XML real, estende a [Migration 19 — Saúde e treinos](doc:migration-19-health) com origem e lote de arquivo; não cria conexão HealthKit, app iOS nem sincronização. Ela não é requisito da primeira migração do James.

## Evolução sem tabelas prematuras

A sequência não obriga construir módulos futuros antes de entregar a base financeira migrável. A preferência e o contexto geográfico por ação são previstos na base/identidade; lugares e visitas/check-ins têm seu contrato de domínio no pacote 13. Veículos, agenda, patrimônio e saúde recebem migrations quando seus marcos começarem. O núcleo de `Document`/versões/vínculos é necessário desde os uploads do financeiro e a conversão do legado (pacote 22); a tela completa da biblioteca, lembretes e OCR podem ser priorizados depois. Conversas, embeddings e memória derivada do assistente do James ficam como extensão futura opcional, sem tabelas ou extensão pgvector obrigatórias no primeiro lançamento.

Referências cruzadas: [Enums](doc:enums), [componentes técnicos](doc:componentes-tecnicos) e [compatibilidade com o 1.x](doc:compatibilidade-legado).
