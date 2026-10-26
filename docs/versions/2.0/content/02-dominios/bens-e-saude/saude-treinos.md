---
id: saude-treinos
title: Saúde e treinos
description: Registros pessoais, rotinas e evolução com unidades e histórico reproduzível.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, agenda-tarefas, ia-e-memoria, migrations
source_refs:
updated: 2026-10-25
---

## Primeira fatia

Planejar catálogo pessoal de exercícios, rotinas e sessões realizadas. A sessão congela a estrutura praticada (exercício, séries, repetições, carga, unidade e observações), para que editar uma rotina não altere os treinos antigos.

Medições corporais são observações com tipo, valor, unidade, data e fonte. Tipos e unidades pertencem a catálogos definidos; valores incompatíveis não são aceitos como JSON livre. Adicionar metas e evolução sem transformar dados anotados em diagnóstico.

## Integrações

Agenda e tarefas podem lembrar um treino; conclusão do lembrete não cria uma sessão realizada. Saúde e diário permanecem privados, fora da IA por padrão. Integração com Apple Health depende de aplicativo/permissões e fica para fase móvel posterior.

## Migração

Não há dados desse módulo no James 1.x. O primeiro desenvolvimento deve começar com seu contrato próprio, sem acrescentar campos nulos de saúde nas tabelas de contatos ou usuários.
