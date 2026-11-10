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
updated: 2026-11-10
---

## Preservar e adaptar

O James atual já tem auditoria global e histórica por registro. A 2.0 preserva essa utilidade e aproveita do SGE a separação entre tabela, histórico contextual, apresentação e consulta escopada. Como os dados são pessoais, nenhum administrador ou outro usuário recebe leitura automática do conteúdo.

## Evento funcional

Guardar espaço, autor humano ou sistema, sujeito, ação, instante e correlação. Mudanças usam allowlist de atributos relevantes; não copiar senha, token, API key, configuração secreta ou texto íntimo completo de diário para o diff. Notas do diário possuem revisões privadas próprias.

Operações financeiras efetivadas registram estorno e substituição com referência ao original. Auditoria não recalcula saldo nem substitui o registro atual do domínio. Job deve indicar sistema e origem (recorrência, importação ou execução de IA) com contexto persistido.

Quando a preferência da pessoa permitir, uma ação de criação/edição iniciada no site pode capturar uma posição pontual do navegador. Guardar esse contexto geográfico numa relação própria por `correlation_id`, fora de `properties` e `attribute_changes`: uma submissão pode gerar vários eventos de auditoria, mas usa uma posição só. O histórico pode mostrar a posição junto à ação; ela não prova que a pessoa esteve no lugar associado ao registro.

Não capturar posição para jobs ou eventos do sistema. Importação confirmada pela pessoa no site pode ter um contexto para a confirmação, nunca um contexto copiado para cada linha importada. Se um job decorrer de uma ação web, guardar a correlação da causa para navegar à submissão humana. O evento do job mantém autoria de sistema. A posição permanece associada somente à ação humana que a capturou.

## Consulta

Histórico contextual dentro do registro e painel com filtros por módulo, sujeito, ação, autor e período. Abrir detalhe e retornar preserva o filtro/página. Resolver nomes e links somente dentro do escopo atual; snapshot mínimo mantém a leitura quando o sujeito foi arquivado/excluído.

## Reversão seletiva de alterações

Oferecer **Reverter campo** em cada linha elegível do diff e **Revisar reversão** no evento, com seleção dos campos e prévia antes da confirmação. A operação usa o valor anterior de cada atributo alterado naquele evento. Campos ausentes do diff conservam seu valor atual, inclusive alterações feitas depois em outros atributos.

Exemplo de uma edição que mudou descrição e valor de R$ 100 para R$ 120, seguida por outra edição somente da data. Reverter o primeiro evento restaura sua descrição anterior e R$ 100, preservando a data atual. Reverter somente a linha do valor preserva também a descrição atual.

A prévia apresenta campo, valor anterior do evento, valor resultante daquele evento e valor atual. Quando o valor atual divergir do resultado registrado no evento, destacar o conflito com uma alteração posterior do mesmo campo. A pessoa pode selecionar conscientemente a restauração do valor antigo ou deixar esse campo como está. Mostrar quando o histórico disponível indicar mudanças posteriores mesmo que o valor tenha voltado a coincidir.

Na confirmação, revalidar proprietário, Policy, estado, revisão atual e allowlist de campos reversíveis dentro da operação transacional. Mudança concorrente após a prévia exige nova revisão da proposta. Aplicar somente os campos selecionados pelas operações normais do módulo, com validação de tipos, relações e invariantes. Relações vinculadas usam a identidade preservada no histórico e exigem conferir existência, escopo e elegibilidade atuais.

A reversão gera uma nova alteração auditada, com autor, campos efetivamente modificados, motivo quando informado, correlação e referência ao evento original. Preservar o histórico original e registrar a operação de forma idempotente. Um campo que já possui o valor desejado é apresentado como tal e dispensado da gravação.

Começar por atributos simples de eventos de edição com valores anteriores completos e confiáveis. Cada domínio define os campos elegíveis e os grupos que precisam ser corrigidos juntos. Campos calculados, segredos, diffs incompletos e transformações legadas sem correspondência segura ficam identificados como indisponíveis para essa ação. Revisões privadas de diário/documentos seguem seu próprio contrato autorizado de histórico.

Valores financeiros efetivados, pagamentos, exclusões e mudanças com efeitos relacionados usam o fluxo de correção ou estorno do módulo. A interface apresenta esse encaminhamento e seus efeitos antes da confirmação. O log fornece a referência histórica, enquanto o domínio coordena pagamentos, faturas, acertos, preços e demais dependências. O detalhamento técnico segue os contratos de [Integrações e consistência](doc:integracoes-e-consistencia).

## Retenção e outros históricos

A convenção atual é retenção vitalícia da auditoria funcional, sem agendar `activitylog:clean`. Mudança disso exige decisão própria. Tentativas de notificações, staging, respostas técnicas de IA e dados geográficos não são Activity Log e têm contratos de retenção separados. Apagar coordenadas expiradas ou a pedido deixa ação, autoria, instante e diff funcional intactos; nunca colocar coordenadas no JSON vitalício do Activity Log.

Tentativas de entrega e uso de IA são registros próprios: não duplicar cada mutação técnica num log genérico sem necessidade. Eliminação de dados pessoais e apagamento dos derivados de IA devem ser projetados junto da política de exclusão de conta antes do lançamento.

## Migração

Importar histórico como legado, preservando data e autoria disponível. Autor antigo não vira uma conta de outro usuário real. Segredos são filtrados e campos sem equivalente permanecem identificados no relatório. O ato de importar gera seu próprio evento com pacote, contagens e responsável, sem gerar milhares de eventos falsos como se o usuário acabasse de criar cada registro manualmente.
