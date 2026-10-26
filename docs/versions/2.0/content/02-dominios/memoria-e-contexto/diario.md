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
updated: 2026-10-25
---

## Experiência inicial

Permitir texto livre em Markdown, título opcional, data do acontecimento, data de escrita, fuso, tags e anexos privados. Uma entrada pode falar de um dia inteiro ou de um instante. Distinguir essa precisão em vez de inventar horário meia-noite para todo registro.

Vincular pessoas, empresas, lugares, visitas, tarefas, compras e eventos de veículo sem copiar seus dados para dentro do texto. Oferecer captura rápida pelo celular e revisão posterior no desktop. Humor e contexto podem ser catálogos opcionais; não exigir formulários longos para escrever uma nota.

## Histórico e privacidade

Revisões preservam o texto que o usuário editou; auditoria registra autoria e evento sem copiar o texto íntimo completo para cada diff. Busca respeita o proprietário e a lixeira. Arquivamento é diferente de exclusão; a política de exclusão definitiva precisa alcançar anexos e índices derivados.

## IA gradual

A IA não reescreve a entrada original. Resumo, título ou fatos extraídos são derivados identificados, com fontes e versão. O envio de diário à IA exige opção própria e fica desligado no início. Uma futura resposta deve diferenciar “o usuário escreveu”, “foi inferido” e “foi confirmado”.

Texto livre e memória estruturada coexistem. Não transformar todo detalhe autobiográfico em um campo obrigatório nem em uma tabela EAV genérica.
