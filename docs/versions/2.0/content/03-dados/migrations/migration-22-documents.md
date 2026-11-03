---
id: migration-22-documents
title: Migration 22 — Documentos e anexos
description: Contrato lógico para arquivos documentais, versões e vínculos compartilhados entre módulos.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, documentos
related: planejamento-2-0, migrations, modelo-de-dados, enums, enum-document-version-origin, documentos-arquivos, migration-00-base, migration-02-records, migration-06-transactions, migration-20-imports
source_refs:
updated: 2026-11-03
---

## `documents`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id` | PK/FK única para `records`; kind `Document`, espaço do mesmo usuário. |
| `title, document_type_label` | Título pesquisável; tipo local opcional, sem lista global obrigatória e sem usar tags como tipo. |
| `document_date, expires_on` | Datas civis opcionais de emissão/assinatura e validade, sem inventar hora. Validade atual é calculada, não copiada em estado mutável. |
| `issuer_party_id, holder_party_id` | FKs opcionais para Party no mesmo espaço, acrescentadas após Migration 03; só por seleção/confirmação humana. |
| `notes, archived_at, deleted_at` | Notas curtas e ciclo de vida; conteúdo integral e segredos não entram em auditoria. |

## `document_versions`

| Campo ou grupo | Contrato |
| --- | --- |
| `id, space_id, document_id, version_number` | Dono e ordem sequencial; UNIQUE(document_id, version_number), espaço igual ao documento. |
| `sha256, origin, uploaded_by_user_id, created_at` | Integridade, origem via [DocumentVersionOrigin](doc:enum-document-version-origin) e autoria/data conhecidas. Hash não implica mesclagem. |
| Relação com `media` | Um objeto `media` privado pertence a uma versão por relação de Media Library. Metadados físicos vêm desse objeto; conteúdo nunca fica no banco relacional. Cada versão tem um original. |

Uma versão não é editada depois de criada. Enviar outra digitalização do mesmo documento cria a próxima versão e a define como corrente pela maior `version_number`; obter um documento renovado, com nova identidade ou validade, cria outro `Document` ligado por `record_links`. Não adicionar boolean `is_current` redundante sem prova de necessidade.

## Vínculos com os módulos

Reutilizar `records` e `record_links` da Migration 02. O registro do documento pode apontar a zero ou vários registros de domínio por tipos de vínculo permitidos em código, como comprovante de transação, contrato de serviço, documento de um bem ou certidão de uma pessoa. Não criar pivôs de anexos independentes por módulo e não aceitar nomes de classe/tabela enviados pelo cliente.

Para uploads no contexto de uma tela, criar documento/versão e vínculo na mesma operação lógica. Selecionar um documento existente cria somente um novo `record_link`. Desvincular um documento nunca apaga o binário, outra versão ou os demais vínculos. Remoção definitiva é operação do módulo Documentos, mostra todos os vínculos afetados e respeita lixeira/retenção.

`fiscal_documents` mantém campos fiscais e de extração, com `source_document_record_id` opcional/único para o arquivo original. O próprio documento tem vínculo com a transação. A nota fiscal estruturada e o seu arquivo são conceitos relacionados, não duas cópias de mídia.

## Integridade, índices e comportamento

- Criar `documents` após `records`, `spaces` e `parties`; adicionar FK de emitente/titular quando Party existir. A migration lógica tem dependências 00, 02 e 03.
- `space_id` é validado em `records`, documento, versão, media e cada ponta do vínculo. Policies protegem biblioteca, busca, seleção, preview e download.
- Um arquivo enviado em lote pelo usuário cria um documento por arquivo. Nunca confiar em `name`, extensão ou `Content-Type` do navegador; validar MIME real, tamanho e arquivos perigosos antes de publicar.
- O download/preview autoriza o documento no servidor. URLs temporárias podem ser assinadas e curtas; sem endereço público permanente. Preview não abre uma exceção de acesso à versão original.
- Índices por `(space_id, expires_on)`, `(space_id, document_date)` e campos de busca usados pela biblioteca. Não indexar texto OCR nem criar busca vetorial nesta migration.
- Checksum pode detectar byte idêntico dentro da conta para avisar a pessoa. Não fazer deduplicação cross-user, união automática ou deleção da origem.
- Remover um `record_link` preserva o documento. Exclusão de documento arquivado/em lixeira não cascata para registros de finanças ou bens; definir purge de versões e binários em política própria e auditável.
- Auditoria guarda metadados mínimos: ids, ação e contexto. Não guardar binários, conteúdo de OCR, números completos de documentos civis, nomes de arquivo íntimos ou texto de certidão em diff/push/log.
- Proteger contra documentos com conteúdo ativo no preview; forçar download seguro para tipos não suportados. Staging temporário e conversões têm expiração própria.

## Conversão do James 1.x

Cada `media` da coleção `attachments` em transações, liquidações e grupos de acerto vira `Document` + primeira `document_version` + um vínculo ao registro legado mapeado. Preservar bytes, hash, MIME, nome, ordem, data disponível, espaço e lixeira. Importar objetos iguais em contextos diferentes sem mesclar; informar ausência ou hash inválido no relatório.

O conversor precisa mapear o `model_type/model_id` legado por uma allowlist; não resolver classes arbitrárias do manifesto. Avatares da coleção separada não são documentos. Anexos não recebem tipo/tag inferido pela descrição. `fiscal_documents` aponta para o mesmo documento original quando os IDs e conteúdo confirmarem a identidade.

## Componentes previstos

CreateDocumentFromUpload; AddDocumentVersion; LinkDocumentToRecord; UnlinkDocumentFromRecord; DocumentPolicy; DownloadDocumentVersion; ValidateDocumentUpload; converter de anexos do James 1.x.
