---
id: enum-document-version-origin
title: Enum — DocumentVersionOrigin
description: Caminho pelo qual um arquivo original entrou numa versão de documento.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, documentos
related: planejamento-2-0, enums, migrations, migration-22-documents, documentos-arquivos
source_refs:
updated: 2026-11-03
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `UserUpload` | `user_upload` | Enviado na área de documentos sem vínculo de origem. |
| `ModuleAttachment` | `module_attachment` | Enviado durante uma operação de domínio e inicialmente ligado a esse registro. |
| `LegacyImport` | `legacy_import` | Convertido de um anexo do James 1.x ou pacote de migração válido. |

## Regras e transições

A origem descreve como o arquivo entrou, não sua validade ou confiança. Relações com transações, acertos e outros módulos ficam em `record_links`; o valor `ModuleAttachment` não prova o vínculo sozinho. Atualizar metadados ou associar o documento a outro Record não muda a origem da versão.
