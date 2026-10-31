---
id: fases-2-0
title: Fases de desenvolvimento
description: Marcos executáveis com dependências, validação de schema e critérios de migração.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: modelo-de-dados, migrations, enums, componentes-tecnicos, compatibilidade-legado, decisoes-e-pendencias, assistente-pessoal-rag, saude-treinos, migration-19-health
source_refs:
updated: 2026-10-31
---

## Ordem proposta

- [ ] **00 — Descoberta e decisões:** inventário 1.x e regras de relatórios; aprovar categorias/tags, recorte do lançamento e decisões em [Pendências](doc:decisoes-e-pendencias).
- [ ] **01 — Provas de modelagem:** validar documento/pagamento/alocação, livro interno e records com compra mista, fatura parcial, reembolso e rateio sem parcela própria. Definir fonte única dos saldos antes de escrever todas as migrations.
- [ ] **02 — Fundação:** starter kit de 2027, Flux 2 gratuito, identidade, espaço pessoal, Policies, tipos monetários, mídia privada e auditoria. [Pacotes 00–02](doc:migrations).
- [ ] **03 — Pessoas e classificação:** contatos/empresas/grupos, tags/categorias, aliases, combobox com criação e regras locais. [Pessoas](doc:pessoas-empresas) e [tags](doc:tags).
- [ ] **04 — Financeiro:** documentos e itens, leitura revisável de XML/QR/imagem, adaptadores fiscais por UF (avaliar o scraper existente do RS), pagador explícito, múltiplos pagamentos, transferências, estornos, relatórios e arquivamento. Provar idempotência e somas sob concorrência. [Finanças](doc:financas).
- [ ] **05 — Cartões e recorrências:** ciclos congelados, parcelas, várias contas por fatura, limite e ocorrências sem duplicatas. [Cartões](doc:cartoes).
- [ ] **06 — Acertos:** participantes/pagadores distintos, parcela própria zero, obrigações e liquidações com integração financeira. [Acertos](doc:acertos).
- [ ] **07 — Comunicação:** central de atenção, preferências individuais, outbox e tentativas, resumos e lembretes que acompanham o domínio. [Notificações](doc:notificacoes).
- [ ] **08 — Migração real:** backport do exportador por navegador no 1.x, pacote consistente, upload no 2.0, prévia para a própria conta e reconciliação. Automatizar os cenários de [compatibilidade](doc:compatibilidade-legado).
- [ ] **09 — IA inicial:** conexões pessoais/instalação explícitas, orçamento e sugestões de tags para transações e itens. Pode propor tags novas, criadas somente após confirmação. Fallback FiscalExtraction apenas quando XML, QR ou leitor determinístico da UF não atender, com permissão por documento e revisão obrigatória. Avaliar à parte TransactionDraft sem nota, sempre iniciando como Draft e sem efetivar pagamentos. Sem classificação de categorias, chat ou ferramentas de pagamento. [IA](doc:ia-e-memoria).
- [ ] **10 — Veículos e organização:** cadastro que não exige bem patrimonial, abastecimento preenchido pela nota e ligado ao item financeiro, pagador explícito, serviços com datas reais, diário genérico, odômetro, checklists, tarefas, agenda local e feed `.ics` somente leitura por URL secreta para iOS. Se aprovada, incluir comandos naturais delimitados para agenda, com alvo validado e confirmação para editar/excluir. [Veículos](doc:veiculos) e [agenda](doc:agenda-tarefas).
- [ ] **11 — Diário e lugares:** notas/revisões, cadastro de lugares e check-in responsivo no próprio site, com localização opcional solicitada pela pessoa. [Diário](doc:diario) e [lugares](doc:lugares-checkins).
- [ ] **12 — Patrimônio e saúde/treinos:** valores e unidades reproduzíveis, sem duplicação financeira; módulos podem ter releases próprios. Começar com registros manuais e a estrutura local de treinos. [Patrimônio](doc:patrimonio) e [saúde](doc:saude-treinos).
- [ ] **12A — Apple HealthKit opcional:** decidir se vale manter um cliente iOS mínimo para sincronização somente de leitura. Primeira fatia: peso/altura e resumo de atividades; depois, amostras de frequência cardíaca e rotas com autorização separada. Sem sincronização em segundo plano no primeiro corte; manter importação XML pelo navegador como alternativa manual. Não é requisito do núcleo migrável e não muda o fluxo de check-in no site. [Contrato de saúde](doc:migration-19-health).
- [ ] **13 — Produtos e preços:** códigos, catálogo pessoal, unidade normalizada, lugares de compra, histórico e cesta pessoal quando houver cobertura suficiente. [Produtos e preços](doc:produtos-precos).
- [ ] **14 — Preparação de produção:** revisão de segurança, performance, acessibilidade, backup/restore, exclusão, atualização, documentação e plano de suporte. Aprovar quais módulos compõem o primeiro lançamento.
- [ ] **15 — Conectores de agenda bidirecionais:** prototipar conta CalDAV do James para Calendário do iPhone e Google Calendar OAuth. Implementar somente após definir conflitos, duplicatas, revogação, exclusões e recuperação; o feed `.ics` continua sendo a integração iOS de leitura.
- [ ] **16 — Descoberta do assistente pessoal:** seguir o [plano do assistente e RAG](doc:assistente-pessoal-rag); escolher fontes opt-in, política de retenção, fronteiras de ferramenta, métricas de qualidade e orçamento; montar conjunto de perguntas/fontes para avaliação. Não requer mudar o primeiro lançamento migrável.
- [ ] **17 — Busca fundamentada (RAG), somente leitura:** prototipar diário e histórico de veículos; comparar SQL, busca textual e PostgreSQL/pgvector; entregar fontes clicáveis, período, filtros por espaço e abstenção sem evidência.
- [ ] **18 — Ferramentas do assistente:** adicionar consultas estruturadas e depois propostas de escrita por módulo, uma ferramenta de cada vez, com Policies, confirmação, idempotência e auditoria. Assistência proativa e voz são fases separadas.

## Checklist por funcionalidade, inspirado no SGE

- [ ] Problema, regra, estados e critérios de aceite descritos.
- [ ] Enum/catálogo escolhido com motivo; migrations e FKs revisadas.
- [ ] Propriedade pessoal validada em consultas, seletores, Action, Policy, arquivo e Job.
- [ ] Transação, concorrência e idempotência definidas onde houver efeitos financeiros/externos.
- [ ] Histórico, exclusão, arquivamento, notificações e IA avaliados sem copiar segredos.
- [ ] UI com estados de erro/vazio/carregamento e captura móvel útil.
- [ ] Mapeamento legado definido e perda/ambiguidade mostrada no relatório.
- [ ] Testes de comportamento, autorização e invariantes; formatação e build pertinentes.
- [ ] Notas Aurelius e catálogos técnicos coerentes com o que realmente foi implementado.

## Exemplos de aceite

Usar os [cenários de validação](doc:cenarios-de-validacao) para avaliar o schema, os protótipos e os fluxos antes da liberação. São critérios planejados, não resultados de testes executados.

## Gate do primeiro lançamento migrável

Conta nova recebe um export real do 1.x pelo navegador, com saldos, dívidas, acertos, tags e arquivos conferidos. Repetição de upload/requisição não duplica dados; usuário A não consulta dados de B por qualquer rota, arquivo, relatório, busca, Job ou resultado de IA. Canais externos ficam inativos até configuração explícita na nova conta.

Nenhuma fase está marcada como implementada apenas porque a documentação foi escrita.
