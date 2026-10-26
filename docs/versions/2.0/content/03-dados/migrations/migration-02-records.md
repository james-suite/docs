---
id: migration-02-records
title: Migration 02 — Registros vinculáveis e proveniência
description: Contrato lógico de tabelas, FKs, índices e conversão: registros vinculáveis e proveniência.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, componentes-tecnicos
source_refs:
updated: 2026-10-25
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 01. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `records`

| Campo ou grupo | Contrato |
| --- | --- |
| `public_id` | ULID único para endereçamento; não substitui autorização. |
| `kind` | RecordKind; tabela/classe resolvida por mapa fechado em código. |
| `revision` | bigint positivo; incrementa em alterações que invalidam derivados. |
| `archived_at, deleted_at` | Metadados de consulta sincronizados com a entidade; tombstone permite invalidar busca e sugestões. |

## `record_link_types`

| Campo ou grupo | Contrato |
| --- | --- |
| `code, label, inverse_label` | Código estável e rótulos pessoais; tipos do sistema usam namespace reservado. |
| `allowed_kinds` | Regra validada por código; restrições de domínio não ficam só em JSON livre. |

## `record_links`

| Campo ou grupo | Contrato |
| --- | --- |
| `from_record_id, to_record_id` | FKs compostas do mesmo espaço. |
| `link_type_id` | FK para tipo permitido. |
| `origin, created_by_user_id` | Fonte da ligação e autoria; inferência de IA precisa de confirmação. |

## Integridade, índices e comportamento

- records guarda identidade/proveniência; dados de finanças, saúde ou diário continuam nas suas tabelas tipadas. Não transformar o sistema em uma tabela universal de atributos.
- Cada entidade vinculável tem record_id único e corresponde a exatamente um kind. Resolver e auditar essa correspondência; não aceitar FQN ou nome de tabela do cliente.
- UNIQUE(space_id, from_record_id, to_record_id, link_type_id); proibir auto-vínculo quando não fizer sentido. Índice inverso para navegar pelos vínculos recebidos.
- Um vínculo não concede acesso nem copia conteúdo. Exclusão/inativação remove derivados e impede resolver conteúdo fora do escopo.

## Conversão do James 1.x

IDs antigos são guardados no mapa da importação, sem reutilizar PKs locais. Vínculos só são criados quando a relação legada é verificável.

## Componentes previstos

RecordResolver; LinkRecords; Policies de origem/destino; invalidação de sugestões e futura busca.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
