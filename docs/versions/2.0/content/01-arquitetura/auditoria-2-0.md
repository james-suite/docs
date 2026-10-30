---
id: auditoria-2-0
title: Auditoria e histórico do James
description: Escopo pessoal, autoria, diffs e distinção entre auditoria funcional e histórico técnico.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: modelo-de-dados, migration-00-base, notificacoes, ia-e-memoria, compatibilidade-legado
source_refs:
updated: 2026-10-29
---

## Preservar e adaptar

O James atual já tem auditoria global e histórica por registro. A 2.0 preserva essa utilidade e aproveita do SGE a separação entre tabela, histórico contextual, apresentação e consulta escopada. Como os dados são pessoais, nenhum administrador ou outro usuário recebe leitura automática do conteúdo.

## Evento funcional

Guardar espaço, autor humano ou sistema, sujeito, ação, instante e correlação. Mudanças usam allowlist de atributos relevantes; não copiar senha, token, API key, configuração secreta ou texto íntimo completo de diário para o diff. Notas do diário possuem revisões privadas próprias.

Operações financeiras efetivadas registram estorno e substituição com referência ao original. Auditoria não recalcula saldo nem substitui o registro atual do domínio. Job deve indicar sistema e origem (recorrência, importação ou execução de IA) com contexto persistido.

Quando a preferência da pessoa permitir, uma ação de criação/edição iniciada no site pode capturar uma posição pontual do navegador. Guardar esse contexto geográfico numa relação própria por `correlation_id`, fora de `properties` e `attribute_changes`: uma submissão pode gerar vários eventos de auditoria, mas usa uma posição só. O histórico pode mostrar a posição junto à ação; ela não prova que a pessoa esteve no lugar associado ao registro.

Não capturar posição para jobs ou eventos do sistema. Importação confirmada pela pessoa no site pode ter um contexto para a confirmação, nunca um contexto copiado para cada linha importada. Se um job decorrer de uma ação web, a localização só pode ser atribuída à ação humana se compartilhar o `correlation_id` original; não usar localização no momento em que o job rodou.

## Consulta

Histórico contextual dentro do registro e painel com filtros por módulo, sujeito, ação, autor e período. Abrir detalhe e retornar preserva o filtro/página. Resolver nomes e links somente dentro do escopo atual; snapshot mínimo mantém a leitura quando o sujeito foi arquivado/excluído.

## Retenção e outros históricos

A convenção atual é retenção vitalícia da auditoria funcional, sem agendar `activitylog:clean`. Mudança disso exige decisão própria. Tentativas de notificações, staging, respostas técnicas de IA e dados geográficos não são Activity Log e têm contratos de retenção separados. Apagar coordenadas expiradas ou a pedido deixa ação, autoria, instante e diff funcional intactos; nunca colocar coordenadas no JSON vitalício do Activity Log.

Tentativas de entrega e uso de IA são registros próprios: não duplicar cada mutação técnica num log genérico sem necessidade. Eliminação de dados pessoais e apagamento dos derivados de IA devem ser projetados junto da política de exclusão de conta antes do lançamento.

## Migração

Importar histórico como legado, preservando data e autoria disponível. Autor antigo não vira uma conta de outro usuário real. Segredos são filtrados e campos sem equivalente permanecem identificados no relatório. O ato de importar gera seu próprio evento com pacote, contagens e responsável, sem gerar milhares de eventos falsos como se o usuário acabasse de criar cada registro manualmente.
