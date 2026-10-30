---
id: lugares-checkins
title: Lugares, visitas e check-ins
description: Histórico de lugares com check-ins feitos no próprio site.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, pessoas-empresas, diario, ia-e-memoria, migrations
source_refs: https://developer.mozilla.org/en-US/docs/Web/API/Geolocation/getCurrentPosition, https://brasilapi.com.br/docs, https://operations.osmfoundation.org/policies/tiles/, https://www.openstreetmap.org/copyright
updated: 2026-10-29
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

## Localização nos registros e no histórico

Como preferência pessoal, a pessoa pode habilitar “Capturar localização ao salvar registros”. Quando habilitada, cada criação, edição ou outra alteração iniciada por ela no site tenta obter uma posição pontual no momento do salvamento. A chamada acontece dentro dessa ação do usuário usando a geolocalização do navegador; não usar `watchPosition`, captura em segundo plano ou localização por IP. O navegador exige permissão explícita e contexto seguro (HTTPS). Se a pessoa negar, a posição expirar, faltar precisão ou o navegador não oferecer o recurso, o registro ainda deve ser salvo normalmente. Permitir desabilitar a captura globalmente e ignorá-la naquele salvamento.

Uma posição de salvamento descreve onde o navegador estava quando a pessoa registrou/alterou algo. Não comprova presença no lugar relacionado ao registro: endereço fiscal, `place_id`, visita/check-in e posição atual são fatos distintos. Tarefas agendadas e alterações automáticas não recebem localização. Uma importação confirmada pela pessoa no site pode ter um contexto para a ação de importação, sem copiar a posição para cada item importado.

Cada ação de formulário tem no máximo uma observação de localização ligada ao `correlation_id` da auditoria. Se salvar uma transação gerar logs para itens e pagamentos, todos apontam para o mesmo contexto; não repetir coordenadas por linha ou model. No histórico do registro, mostrar a posição, horário e precisão junto à ação correspondente. Uma posição imprecisa deve aparecer como área/círculo de precisão, não como um ponto exato inventado; se o navegador não fornecer precisão, indicar que ela é desconhecida.

O histórico geográfico fica em armazenamento próprio, referenciado pela correlação da ação e separado das propriedades do Activity Log. A auditoria funcional tem retenção vitalícia no plano atual, mas isso não pode impor retenção vitalícia a coordenadas. A pessoa precisa poder consultar, exportar e apagar dados de localização; definir o prazo padrão e opções de retenção antes de fechar o schema. A posição não entra em IA por padrão nem fica acessível a administradores da instalação.

### Mapa com OpenStreetMap

O histórico pode exibir as ações num mapa privado, com filtros por período, módulo e tipo de ação e agrupamento de pontos próximos. A visualização pode abrir o registro e o evento de auditoria correspondente, respeitando o mesmo escopo da conta. Tiles são carregados somente quando alguém abre o mapa; queda do serviço de mapas não afeta salvar registros nem consultar o texto do histórico.

Usar dados OpenStreetMap com atribuição visível “© OpenStreetMap contributors”. O servidor público `tile.openstreetmap.org` tem política de uso, capacidade limitada e disponibilidade sem SLA; não tratá-lo como API gratuita ilimitada de produção. Tornar o provedor de tiles configurável e escolher provedor compatível ou hospedagem própria conforme o volume. Respeitar cache e proibição de pré-carregar áreas em lote. Não enviar descrição do registro, nome da pessoa ou metadados da auditoria ao provedor; pedidos de tiles ainda revelam a área do mapa que está sendo visualizada.

## Dados e privacidade

Histórico de localização é privado e não entra em IA por padrão. O usuário pode exportar, corrigir e apagar visitas e contextos geográficos de auditoria. A captura é opcional, tem finalidade explicada na preferência e respeita a permissão do navegador. Localização exata pode revelar casa, saúde, trabalho e rotina; aplicar autorização de leitura, retenção própria e exclusão sem apagar o evento funcional da auditoria.

Usar latitude/longitude com limites e índices de data no início. PostGIS e particionamento devem ser adotados quando houver requisito de consultas espaciais ou volume que os justifique. Não presumir integração automática com o histórico do Google nem um formato estável de exportação sem verificar o arquivo disponível.
