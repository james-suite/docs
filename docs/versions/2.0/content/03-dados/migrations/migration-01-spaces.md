---
id: migration-01-spaces
title: Migration 01 — Identidade e espaço pessoal
description: Contrato lógico de tabelas, FKs, índices e conversão: identidade e espaço pessoal.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, arquitetura-2-0, componentes-tecnicos
source_refs:
updated: 2026-10-29
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 00. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `spaces`

| Campo ou grupo | Contrato |
| --- | --- |
| `owner_user_id` | FK users, NOT NULL e UNIQUE; um espaço pessoal por conta na primeira versão. |
| `name` | Nome de exibição opcional; não é identificador de autorização. |
| `currency, timezone` | Moeda padrão e fuso IANA. |
| `archived_at` | Indisponibilidade do espaço; não elimina dados. |

## Integridade, índices e comportamento

- Não criar memberships, convites de workspace ou papéis colaborativos agora: o usuário não prevê relação entre contas reais.
- space_id sempre vem da conta autenticada; importação não pode escolher proprietário pelo arquivo.
- Cada tabela escopada tem UNIQUE(space_id, id), permitindo FKs compostas entre registros de um mesmo espaço. Policies e Actions também revalidam o escopo.
- Adicionar FKs de space_id em media e activity_log após criar spaces. Criar então `activity_location_contexts` como extensão opcional de `activity_log`, com escopo e `correlation_id` único; não colocar coordenadas no JSON de auditoria.
- A retenção de `activity_location_contexts` é definida separadamente da retenção vitalícia do Activity Log; apagar o contexto geográfico preserva a ação auditada.
- Jobs usam IDs persistidos e nunca auth() como fonte de contexto. Eventos automáticos não recebem localização da pessoa que os iniciou em outro momento.

## Conversão do James 1.x

A conta autenticada no 2.0 é a dona do arquivo importado. O usuário do 1.x aparece somente como proveniência histórica quando necessário.

## Componentes previstos

PersonalSpaceContext; criação idempotente do espaço; Policies por proprietário; casts de timezone e moeda.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
