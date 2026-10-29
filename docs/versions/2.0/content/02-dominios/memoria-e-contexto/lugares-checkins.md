---
id: lugares-checkins
title: Lugares, visitas e check-ins
description: Histórico de lugares com check-ins feitos no próprio site.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, pessoas-empresas, diario, ia-e-memoria, migrations
source_refs: https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/getCurrentPosition, https://brasilapi.com.br/docs
updated: 2026-10-28
---

## Começar por eventos compreensíveis

Separar lugar (places) e visita/check-in (visits). Um lugar pode ser uma empresa/filial, casa ou local livre. A visita registra quando e como a presença foi informada e pode guardar uma posição pontual, com sua precisão, quando disponível. Registrar “estive na oficina” já tem valor sem coordenadas.

Guardar início/fim, fuso observado, precisão quando houver, origem da captura e data de recebimento. Uma compra ou descrição de cartão pode sugerir o local, mas não prova que a pessoa esteve lá. Inferências ficam pendentes de confirmação.

Um lugar também pode representar um estabelecimento de uma empresa, com identificador de filial e endereço fiscal. Uma nota pode sugerir o posto/mercado exato, mas isso só atualiza a relação da compra com o lugar após correspondência confiável ou revisão. Registrar o endereço do estabelecimento não exige inventar latitude/longitude nem criar uma visita.

### Preenchimento de endereço por CEP

No cadastro/edição de um lugar, oferecer consulta por CEP para sugerir logradouro, bairro, município e UF. A versão 2 da consulta pode retornar coordenadas derivadas de CEP/OSM; tratá-las como aproximadas e opcionais, nunca como ponto exato do estabelecimento ou como check-in. Mostrar origem e permitir correção. Uma falha da consulta não bloqueia o preenchimento manual.

## Check-in no site

O fluxo acontece no próprio site, com layout responsivo para celular e desktop. A pessoa escolhe um lugar conhecido ou informa um novo e, se quiser, toca em “Usar minha localização” para preencher uma posição naquele check-in. O navegador solicita a permissão nesse momento; se a pessoa negar ou a posição não estiver disponível, o cadastro manual continua funcionando.

O site guarda a hora do evento, fuso observado, origem do cadastro e posição opcional. A localização só é solicitada pela ação explícita no formulário. Em produção, o site precisa estar em HTTPS para o navegador disponibilizar a geolocalização.

## Dados e privacidade

Histórico de localização é privado e não entra em IA por padrão. O usuário pode exportar, corrigir e apagar visitas. A captura da posição é opcional, tem finalidade explicada junto ao botão e respeita a permissão do navegador.

Usar latitude/longitude com limites e índices de data no início. PostGIS e particionamento devem ser adotados quando houver requisito de consultas espaciais ou volume que os justifique. Não presumir integração automática com o histórico do Google nem um formato estável de exportação sem verificar o arquivo disponível.
