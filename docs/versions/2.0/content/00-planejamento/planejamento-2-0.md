---
id: planejamento-2-0
title: Planejamento do James
description: Índice de produto, domínio, schema, enums, componentes e migração.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: home, modelo-de-dados, migrations, enums, escopo-2-0, fases-2-0, decisoes-e-pendencias, assistente-pessoal-rag, modulos-e-integracoes
source_refs:
updated: 2026-10-31
---

## Direção

O planejamento preserva a experiência e os dados úteis do James, melhora o núcleo financeiro e prepara módulos pessoais mais conectados. Esta é a documentação principal; a documentação da versão 1.x fica disponível no seletor de versões e na página inicial. Desenvolvimento previsto para 2027 com starter kit disponível naquele momento e Flux 2 gratuito.

Cada conta terá seu próprio conjunto de dados. Relação entre usuários reais não está prevista inicialmente; rateios são registros privados com contatos. A migração será por arquivo baixado do 1.x e enviado no navegador do 2.0, sempre para a conta autenticada.

## Planejamento técnico

| Leitura | Responsabilidade |
| --- | --- |
| [Escopo](doc:escopo-2-0) | Núcleo migrável, módulos planejados e recorte de releases. |
| [Arquitetura](doc:arquitetura-2-0) | Limites do monólito e propriedade pessoal. |
| [Modelo de dados](doc:modelo-de-dados) | Tipos, cardinalidades, dinheiro e invariantes. |
| [Migrations](doc:migrations) | 21 pacotes lógicos com tabelas, campos, FKs, índices e conversão. |
| [Enums](doc:enums) | Estados/conjuntos fechados com valores persistidos e regras. |
| [Componentes técnicos](doc:componentes-tecnicos) | Models, Actions, Policies, casts, Jobs e componentes de interface. |
| [UI e UX](doc:ui-ux-2-0) | Flux gratuito e padrões adaptados do SGE. |
| [Auditoria](doc:auditoria-2-0) | Autoria, escopo e histórico funcional. |
| [Migração pelo navegador](doc:migracao-2-0) | Arquivo completo, prévia, confirmação e destino pessoal. |
| [Compatibilidade 1.x](doc:compatibilidade-legado) | Conversão por entidade e reconciliação. |
| [Cenários de validação](doc:cenarios-de-validacao) | Exemplos numéricos e resultados que o desenho precisa suportar. |
| [Fases](doc:fases-2-0) | Sequência de protótipos, implementação e preparação do lançamento. |
| [Decisões e pendências](doc:decisoes-e-pendencias) | Confirmado, proposto e ainda em aberto. |
| [Módulos candidatos e integrações](doc:modulos-e-integracoes) | Oportunidades futuras e fluxos entre módulos sem duplicar dados. |

## Domínios detalhados

As notas de domínio seguem pastas por contexto: [finanças](doc:financas), [organização pessoal](doc:tags), [memória e contexto](doc:diario) e [bens e saúde](doc:veiculos). A visão do [assistente pessoal do James](doc:assistente-pessoal-rag) é uma evolução futura separada das funções iniciais de IA. A página inicial lista os módulos e mostra o mapa completo das pastas.

## Princípios

Manter o produto reconhecível; separar fato econômico e pagamento; não contar fatura ou reembolso duas vezes; permitir parcela própria zero; classificar sem criar tags duplicadas; preservar datas e procedência; não criar usuários a partir de contatos; usar IA de forma opcional e revisável; não armazenar indiscriminadamente tudo em JSON ou copiar o banco inteiro para um modelo.

As notas de migrations e enums são contratos propostos. O livro interno e o registro comum de vínculos passam por protótipo antes de se tornarem decisões definitivas. O formato de migração deve ser estável antes do primeiro lançamento, e não acrescentado só no fim.
