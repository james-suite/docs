---
id: migration-13-places
title: Migration 13 — Lugares e check-ins
description: Contrato lógico de tabelas, FKs, índices e conversão: lugares e check-ins.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, lugares-checkins, pessoas-empresas, produtos-precos, componentes-tecnicos
source_refs:
updated: 2026-10-26
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 03. O pacote permite associar uma empresa aos seus estabelecimentos e resolver uma filial a partir de identificadores fiscais. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `places`

| Campo ou grupo | Contrato |
| --- | --- |
| `name, party_id, address` | Lugar pessoal, empresa/filial opcional e endereço estruturado. |
| `latitude, longitude, timezone` | Coordenadas opcionais com limites; fuso IANA. |

## `visits`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, place_id, started_at, ended_at` | Visita; fim opcional não anterior ao início. |
| `latitude, longitude, precision_meters, observed_timezone` | Posição pontual opcional e qualidade observada no check-in. Coordenadas devem ser ambas nulas ou válidas. |
| `source, received_at, notes` | Origem do formulário do site, hora de gravação no servidor e observação privada. |

## place_identifiers

| Campo ou grupo | Contrato |
| --- | --- |
| place_id, scheme, country, value_normalized | Identificador de uma unidade/filial, como CNPJ completo, com país e esquema. |
| source, verified_at | Procedência e revisão da correspondência. |

Unicidade por espaço, esquema, país e valor normalizado. Um estabelecimento não exige coordenadas para ser útil: endereço e identificador fiscal podem bastar para reconhecer o posto ou mercado. Geocodificação, se oferecida, é uma etapa separada e sua fonte/data precisam ser guardadas.

Após criar places, acrescentar a FK composta de fiscal_documents.issuer_place_id, campo previsto pelo pacote 06. A relação é opcional e usa a empresa emitente já resolvida; não converter endereço em coordenadas sem uma fonte registrada.

## Integridade, índices e comportamento

- CHECK latitude -90..90 e longitude -180..180, ambas presentes ou ambas nulas. Precisão não negativa.
- Visitas, lugares e registros associados pertencem ao mesmo espaço; a Policy valida cada referência no envio do formulário.
- Presença inferida de compra não entra como visita confirmada. A origem da captura é definida pelo fluxo do próprio site.
- Uma empresa pode ser dona/operadora de vários lugares; transação e nota fiscal podem apontar para empresa e unidade separadamente. Resolver filial por identificador/endereço, e pedir revisão quando a correspondência for ambígua.

## Conversão do James 1.x

Não há visitas estruturadas para migrar. Notas antigas podem ser vinculadas manualmente; não geocodificar toda a vida financeira sem consentimento.

## Componentes previstos

CreateVisit; BrowserLocationCapture; PlacesSearch; seleção/criação de lugar no formulário.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
