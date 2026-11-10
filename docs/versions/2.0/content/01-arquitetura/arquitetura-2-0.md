---
id: arquitetura-2-0
title: Arquitetura do James
description: Aplicação modular com isolamento pessoal, operações explícitas e base para memória futura.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: modelo-de-dados, componentes-tecnicos, enums, migrations, ui-ux-2-0, ia-e-memoria, assistente-pessoal-rag, decisoes-e-pendencias
source_refs:
updated: 2026-11-08
---

## Direção confirmada

Laravel com Livewire, Blade, Flux 2 gratuito e o starter kit oficial disponível quando o desenvolvimento começar em 2027. Versões de PHP, Laravel, Livewire e pacotes serão verificadas naquele momento. Flux Pro não faz parte do requisito.

O produto mantém os módulos reconhecíveis do James atual e reorganiza a persistência que limita múltiplos pagamentos, cartões e histórico. Código de domínio validado pode inspirar a implementação; a reescrita não é motivo para abandonar regras úteis.

## Multiusuário inicial

Cada conta tem dados pessoais isolados. Relacionamento entre contas reais não está previsto inicialmente. O Space é o limite interno pessoal 1:1, sem memberships nem compartilhamento na primeira versão.

Contato, empresa e participante de rateio pertencem ao catálogo do próprio usuário. Não vincular contas reais automaticamente por e-mail/documento. Dividir despesas com parcela própria zero funciona dentro do próprio histórico.

## Organização modular

Usar monólito Laravel modular por responsabilidades, mantendo transações locais para operações que precisam de atomicidade. Não introduzir microserviços ou motor universal de workflows. CRUD simples segue convenções do projeto; pagamentos, rateios, importações, classificações e notificações justificam Actions/Services específicas.

Fundação comum: identidade/escopo, arquivos privados, auditoria, tags, unidades/moedas, notificações, filas e portabilidade. Módulos mantêm suas tabelas e regras; relacionamentos explícitos conectam financeiro, pessoas, empresas, veículos, diário e lugares.

## Fronteiras propostas

- Documento financeiro descreve o fato; pagamento/alocação descreve a quitação; saldo não depende de tag especial.
- Cartão representa passivo com ciclos/parcelas; pagar fatura não é outra despesa.
- Rateio distingue consumidores e pagadores; nenhum acesso a outra conta é necessário.
- Catálogo de pessoas e organizações oferece identidade reutilizável para fornecedor, prestador, contato e lugar.
- Notificações têm outbox e tentativas; leitura interna não se mistura com transporte externo.
- IA é opcional e restrita à classificação de tags específicas para transações/itens e ao fallback fiscal autorizado. No modo Suggest, a pessoa aplica o resultado completo; tags ausentes são criadas na mesma operação, sem confirmação individual. O modo automático exige autorização própria. Categorias financeiras não são classificadas por IA.
- O assistente pessoal do James é uma evolução posterior: começa com busca somente leitura e respostas ligadas a fontes; ferramentas e ações chegam por módulo, com permissões e confirmações próprias. Não amplia o escopo do primeiro lançamento migrável.
- Diário e check-ins enriquecem a memória pessoal por registros no site; a localização é capturada somente em ações iniciadas pela pessoa, conforme a preferência e a permissão do navegador. Não há check-in automático ou monitoramento contínuo.

## Gates de modelagem

O [modelo de dados](doc:modelo-de-dados) propõe um registro de entidades vinculáveis e um livro financeiro interno. Validar ambos em um protótipo pequeno de pagamento misto, fatura parcial, rateio sem parcela própria e importação. Se não passarem pelo critério de simplicidade e reconciliação, simplificar antes de criar o banco definitivo.

## Contratos transversais

Servidor determina proprietário; Models, consultas, Livewire, filas, auditoria, anexos, busca e importações respeitam o mesmo escopo. Revalidar IDs e estado dentro das transações críticas. Banco garante unicidade e FKs; cache e interface são auxiliares.

Dinheiro e quantidades usam aritmética exata. Datas de negócio e instantes de captura são diferentes. Arquivamento preserva histórico; exclusão de rascunho é diferente de estorno de operação efetivada.

Number padroniza a apresentação numérica e monetária. Carbon atende datas e cálculos temporais. Locale, moeda e fuso são resolvidos pelo contexto comum, com fontes e precedência definidas em [Componentes técnicos](doc:componentes-tecnicos). Todos os módulos reutilizam esse contrato.

## Reuso entre projetos

A matriz em [Componentes técnicos](doc:componentes-tecnicos) registra os candidatos concretos do SGE e do James atual, o modo de reaproveitamento e os limites de cada peça. A regra geral é portar comportamentos com contrato útil e adaptar ao domínio pessoal do James; não importar regras acadêmicas do SGE nem carregar limitações do modelo 1.x. Só extrair uma biblioteca compartilhada quando dois projetos precisarem do mesmo contrato estável e a dependência puder ser mantida e testada sem acoplamento aos domínios.

Adotar Scout com Meilisearch para busca textual e seletores pesquisáveis, aproveitando a integração do SGE. Consultas estruturadas, integridade e cálculos continuam no PostgreSQL/Eloquent. O contrato de escopo e indexação está em [Componentes técnicos](doc:componentes-tecnicos). A [visão do assistente pessoal](doc:assistente-pessoal-rag) mantém a recuperação semântica em fase posterior, com fontes tipadas e uma fronteira pequena para provedores.

## Contratos compartilhados de integração

Donos dos fatos, publicação recuperável, revisões, idempotência, limites de somas e propagação de correções estão centralizados em [Integrações e consistência](doc:integracoes-e-consistencia). As notas de cada módulo definem somente os efeitos específicos. Módulos de agenda, tarefas e patrimônio recebem conexões opcionais por extensões, conforme a matriz de dependências.
