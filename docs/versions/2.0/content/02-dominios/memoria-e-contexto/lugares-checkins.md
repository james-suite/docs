---
id: lugares-checkins
title: Lugares, visitas e check-ins
description: Histórico de lugares com captura manual, importação e integração gradual com iOS.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, pessoas-empresas, diario, ia-e-memoria, migrations
source_refs: https://support.apple.com/guide/shortcuts/apd8ebfc4e8e/ios, https://developer.apple.com/documentation/corelocation/handling-location-updates-in-the-background
updated: 2026-10-26
---

## Começar por eventos compreensíveis

Separar lugar (`places`), visita/check-in (`visits`) e amostra bruta de coordenada. Um lugar pode ser uma empresa/filial, casa ou local livre. A visita registra quando e como a presença foi informada. Coordenadas podem ser opcionais; registrar “estive na oficina” já tem valor mesmo sem GPS.

Guardar início/fim, fuso observado, precisão quando houver, origem da captura e data de recebimento. Uma compra ou descrição de cartão pode sugerir o local, mas não prova que a pessoa esteve lá. Inferências ficam pendentes de confirmação.

Um lugar também pode representar um estabelecimento de uma empresa, com identificador de filial e endereço fiscal. Uma nota pode sugerir o posto/mercado exato, mas isso só atualiza a relação da compra com o lugar após correspondência confiável ou revisão. Registrar o endereço do estabelecimento não exige inventar latitude/longitude nem criar uma visita.

## Entrega inicial

Check-in manual pelo navegador, seleção de lugar conhecido, captura de posição atual mediante ação/permissão do usuário e importação de arquivo em formato explicitamente suportado. Um atalho no iPhone pode enviar um check-in a uma API do James, usando token revogável restrito a registrar visitas e sem acesso aos demais dados.

O endpoint aceita identificador de evento do dispositivo para deduplicar repetições, distingue hora observada de hora recebida e rejeita referências de outro espaço. O formato inicial suporta um evento por requisição; lote/offline pode vir depois. Não prometer detecção contínua de localização apenas pelo site/PWA.

## Automação no iPhone

Atalhos da Apple oferece gatilhos de chegada/saída e automações pessoais; é um caminho a validar para lugares selecionados. Isso não equivale a uma linha do tempo contínua de todos os deslocamentos. Rastreamento em segundo plano exige projeto móvel próprio, permissões e avaliação do comportamento do Core Location, bateria e disponibilidade.

Fontes oficiais: [automações pessoais do Atalhos](https://support.apple.com/guide/shortcuts/apd690170742/ios), [gatilhos de viagem](https://support.apple.com/guide/shortcuts/apd8ebfc4e8e/ios) e [Core Location em segundo plano](https://developer.apple.com/documentation/corelocation/handling-location-updates-in-the-background). Reavaliar limitações e versão do iOS em 2027.

## Dados e privacidade

Histórico de localização é privado e não entra em IA por padrão. O usuário pode exportar, corrigir e apagar visitas e revogar dispositivos. Captura bruta contínua fica fora do schema inicial; quando existir, terá retenção própria e não será duplicada indefinidamente em logs.

Usar latitude/longitude com limites e índices de data no início. PostGIS e particionamento devem ser adotados quando houver requisito de consultas espaciais ou volume que os justifique. Não presumir integração automática com o histórico do Google nem um formato estável de exportação sem verificar o arquivo disponível.
