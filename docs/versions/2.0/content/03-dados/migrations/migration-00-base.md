---
id: migration-00-base
title: Migration 00 — Base técnica do starter kit
description: Contrato lógico de tabelas, FKs, índices e conversão: base técnica do starter kit.
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

Dependências lógicas: starter kit e pacotes selecionados em 2027. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `users`

| Campo ou grupo | Contrato |
| --- | --- |
| `Campos nativos` | Conservar autenticação e segurança fornecidas pelo starter kit escolhido em 2027; não fixar schema desatualizado agora. |
| `is_instance_admin` | boolean false; administração operacional de contas, sem acesso automático aos dados pessoais. |

## `user_preferences`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id` | FK users, única. |
| `locale, timezone` | Locale e fuso IANA; valores explícitos. |
| `default_currency` | Código ISO de moeda; preferência inicial do espaço, sem conversão automática de histórico. |
| `capture_location_on_save` | Boolean false por padrão, por ser um dado sensível num sistema multiusuário; ao ativar, tenta capturar uma posição do navegador em cada mutação manual no site. A permissão do navegador continua obrigatória. Ausência, recusa ou timeout curto não bloqueiam nem atrasam indefinidamente o salvamento. |

## `cache / jobs / failed_jobs / sessions`

| Campo ou grupo | Contrato |
| --- | --- |
| `Estrutura nativa` | Usar migrations dos pacotes selecionados; não duplicar a infraestrutura. |

## `media`

| Campo ou grupo | Contrato |
| --- | --- |
| `Estrutura do pacote` | Arquivos privados, autorização pelo registro e espaço; conteúdo binário fora do banco. |
| `space_id` | Coluna adicional e índice; garantir mesmo proprietário do Model associado. |

## `activity_log`

| Campo ou grupo | Contrato |
| --- | --- |
| `Estrutura do pacote` | Adaptar a versão efetiva do Spatie em 2027. |
| `space_id, correlation_id` | Escopo persistido e correlação; autoria usa causer_type/causer_id nativos, com User como autor humano e nulo para sistema. |
| `properties.source_snapshot, attribute_changes` | Usar campos nativos para snapshot mínimo e diff permitido, sem duas fontes de mudanças; sem segredos ou texto íntimo completo. |

## Contrato lógico: `activity_location_contexts`

Tabela separada e opcional, criada após `spaces` no pacote 01 e ligada à submissão por `space_id` e `correlation_id` único. Guardar `user_id`, `latitude`, `longitude`, `accuracy_meters` nullable, `captured_at` e `source` (`BrowserLocation`). Uma ação do usuário recebe no máximo um contexto, ainda que gere várias linhas no `activity_log`. Coordenadas e precisão nunca entram em `activity_log.properties` ou diffs permanentes. Indexar por espaço/horário para a consulta histórica; avaliar índice espacial só quando o volume ou as consultas justificarem.

## Integridade, índices e comportamento

- Não executar migrations do 1.x no banco novo. Baseline dos pacotes precede as extensões de domínio.
- Auditoria funcional tem retenção vitalícia como na convenção atual; qualquer mudança nessa política exige decisão própria. Logs técnicos, arquivos temporários e respostas de IA têm retenção independente.
- Contextos geográficos têm retenção e exclusão independentes do Activity Log. Definir o prazo/opções antes do schema final; expirar ou apagar a localização não apaga a autoria, ação ou diff funcional.
- Obter coordenadas somente durante uma submissão/confirmação iniciada pela pessoa, com permissão do navegador; sem IP fallback, tracking em segundo plano ou posição inventada para jobs. Uma importação confirmada pode receber um contexto único, sem replicá-lo em cada item. Falha da geolocalização não impede o salvamento.
- Contextos geográficos têm retenção e exclusão independentes do Activity Log. Definir o prazo/opções antes do schema final; expirar ou apagar a localização não apaga a autoria, ação ou diff funcional.
- Obter coordenadas somente durante uma submissão iniciada pela pessoa, com permissão do navegador; sem IP fallback, tracking em segundo plano ou posição inventada para jobs/imports. Falha da geolocalização não impede o salvamento.
- Mídia e auditoria recebem as FKs de spaces no pacote 01, depois de a tabela existir. Não criar FK para tabela futura neste pacote.

## Conversão do James 1.x

Não importar senhas, sessões, tokens, 2FA ou flags de administração da instalação antiga. Mídia e histórico têm conversores próprios e preservam origem.

## Componentes previstos

Policies de arquivos/auditoria; helpers de moeda e data; jobs com contexto explícito; integração dos pacotes.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
