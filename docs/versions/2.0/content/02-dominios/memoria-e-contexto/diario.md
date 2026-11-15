---
id: diario
title: Diário e memória autobiográfica
description: Notas pessoais com datas, revisões, anexos e relações com eventos reais.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, lugares-checkins, ia-e-memoria, tags, migrations
source_refs:
updated: 2026-11-14
---

## Vários diários

A pessoa pode criar quantos diários quiser, com nome e descrição opcional, para organizar diferentes assuntos ou tipos de escrita. Cada diário pertence ao espaço pessoal por FK e contém suas entradas. A criação de uma entrada permite escolher o diário, preservando o rascunho ao trocar a seleção. A interface oferece visão consolidada e filtro por diário, além de mover entradas entre diários do próprio espaço.

Arquivar um diário conserva suas entradas e revisões, mantendo consulta histórica e restauração. A exclusão de um diário com entradas exige escolher entre realocação e exclusão explícita do conteúdo pelo fluxo próprio. Exportação e importação preservam os diários, suas entradas, revisões e vínculos. O catálogo começa conforme a criação pela pessoa, com nome livre e sem limite funcional arbitrário de quantidade.

## Experiência inicial

Permitir texto livre em Markdown, título opcional, data do acontecimento, data de escrita, fuso, tags e vínculos a documentos privados. Uma entrada pode falar de um dia inteiro ou de um instante. Distinguir essa precisão em vez de inventar horário meia-noite para todo registro. Enviar um arquivo pelo diário cria um Document; ele aparece também na biblioteca.

Vincular pessoas, empresas, lugares, visitas, tarefas, compras e eventos de veículo sem copiar seus dados para dentro do texto. Oferecer captura rápida pelo celular e revisão posterior no desktop. Humor e contexto podem ser catálogos opcionais; não exigir formulários longos para escrever uma nota.

## Histórico e privacidade

Revisões preservam o texto que o usuário editou; auditoria registra autoria e evento sem copiar o texto íntimo completo para cada diff. Busca respeita o proprietário e a lixeira. Arquivamento é diferente de exclusão; a política de exclusão definitiva precisa alcançar anexos e índices derivados.

## IA gradual

A IA não reescreve a entrada original. Resumo, título ou fatos extraídos são derivados identificados, com fontes e versão. O envio de diário à IA exige opção própria e fica desligado no início. Uma futura resposta deve diferenciar “o usuário escreveu”, “foi inferido” e “foi confirmado”.

### Referência de escrita e conteúdo para e-mails

Na evolução do assistente, textos dos diários podem servir de referência para rascunhos de e-mail e outras tarefas de escrita autorizadas. Configurar por diário as finalidades permitidas, com uso de estilo e uso de conteúdo pessoal independentes e desligados inicialmente. A seleção de um diário para escrita pode ser refinada por entrada ou período na tarefa atual.

Para estilo, extrair características como tom, vocabulário, ritmo, formalidade e organização a partir de trechos selecionados. Um perfil derivado é revisável, possui fontes/revisões e acompanha a revogação das fontes. Para conteúdo, recuperar os fatos pertinentes ao pedido e apresentar quais entradas contribuíram para o rascunho. A autorização de estilo precisa preservar a confidencialidade dos fatos íntimos presentes nos exemplos. Oferecer prévia dos trechos usados e permitir remover fontes antes de enviar ao provedor de IA.

O rascunho pode combinar a conversa de e-mail e as fontes de diário autorizadas para aquela tarefa. A revisão permite conferir destinatários, texto e anexos antes do envio. Edições, exclusões, mudança de diário e revogação atualizam a elegibilidade e invalidam índices/perfis derivados afetados. O fluxo completo de e-mail fica como integração futura em avaliação, descrita em [Assistente pessoal e RAG](doc:assistente-pessoal-rag).

Texto livre e memória estruturada coexistem. Não transformar todo detalhe autobiográfico em um campo obrigatório nem em uma tabela EAV genérica.
