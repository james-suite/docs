---
id: migration-03-parties
title: Migration 03 — Pessoas, empresas e grupos
description: Contrato lógico de tabelas, FKs, índices e conversão: pessoas, empresas e grupos.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, pessoas-empresas, componentes-tecnicos
source_refs: https://brasilapi.com.br/docs
updated: 2026-10-28
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `parties`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, kind` | Registro único e PartyKind. |
| `display_name, legal_name` | Nome de uso obrigatório; razão social opcional. |
| `notes_markdown, archived_at` | Notas privadas e arquivamento. |

## `party_identifiers`

| Campo ou grupo | Contrato |
| --- | --- |
| `party_id, scheme, country, value_normalized` | Documento/identificador normalizado. Unicidade por parte e esquema, sem unicidade global entre espaços. |
| `source_code, retrieved_at, verified_at` | Procedência opcional do identificador consultado em fonte pública, como BrasilAPI; revisão humana permanece distinta da data de consulta. |

## `party_contact_methods`

| Campo ou grupo | Contrato |
| --- | --- |
| `party_id, kind, label, value, is_primary` | Métodos tipados como telefone/e-mail/site; vários por pessoa/empresa. Campos de formato validados. |

## `party_groups / party_group_members`

| Campo ou grupo | Contrato |
| --- | --- |
| `name, normalized_name / group_id, party_id` | Grupos pessoais e associação única por grupo/parte. |

## `party_relationship_types`

| Campo ou grupo | Contrato |
| --- | --- |
| `code, label, inverse_label` | Catálogo pessoal para prestador, funcionário e relações similares. |

## `party_relationships`

| Campo ou grupo | Contrato |
| --- | --- |
| `from_party_id, to_party_id, type_id` | Relação dirigida dentro do espaço. |
| `valid_from, valid_until, notes` | Vigência opcional, fim não anterior ao início. |
| `origin, source_reference_id, confidence` | Origem manual/externa, referência opcional e grau de confiança; uma sugestão externa não equivale a relação confirmada. |
| `verified_by_user_id, verified_at` | Revisão e autoria da confirmação quando existente. |

## party_external_references

| Campo ou grupo | Contrato |
| --- | --- |
| party_id, source_code, external_id, country | Identidade da parte nesta fonte auxiliar, sem usar o identificador externo como PK local. |
| entity_type, observed_at, source_version | Tipo da entidade, instante de consulta e versão da base/processador. |
| field_snapshot, snapshot_schema_version | Campos externos selecionados para revisão, com payload permitido e versão explícita; não armazenar raspagem arbitrária sem limite. |
| verified_at, verified_by_user_id | Confirmação explícita das informações aplicadas ao cadastro. |

## Integridade, índices e comportamento

- Nome semelhante ou documento igual gera sugestão de duplicidade, nunca fusão automática.
- Uma relação externa aponta para sua referência de origem e continua pendente até revisão. Atualização da fonte não substitui silenciosamente os dados locais nem altera vínculos financeiros.
- source_reference_id em party_relationships é FK composta opcional para party_external_references do mesmo espaço.
- Índices por espaço/nome normalizado e por documento pesquisável. Máscaras são apresentação, não persistência.
- Preservar grupos e relações após arquivar partes. Exclusão física proibida quando referenciada em financeiro ou histórico.

## Conversão do James 1.x

Contact → Party(Person); ContactGroup e pivô preservados. Notas, contatos e mídia são copiados. Alteração para Organization é uma revisão posterior.

## Componentes previstos

Create/UpdateParty; MergeParties sob revisão; normalização de documentos e contatos; combobox com criação inline autorizada.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
