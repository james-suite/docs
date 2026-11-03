---
id: enum-fiscal-capture-method
title: Enum — FiscalCaptureMethod
description: Origem usada para capturar um documento fiscal no James.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas, migration-06-transactions
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `QrCode` | `qr_code` | Leitura de chave ou código de acesso da nota. |
| `XmlUpload` | `xml_upload` | Arquivo fiscal estruturado fornecido pelo usuário. |
| `ImageScan` | `image_scan` | Foto ou imagem processada por OCR. |
| `ManualEntry` | `manual_entry` | Registro digitado sem arquivo de origem. |
| `LegacyImport` | `legacy_import` | Documento preservado por importação de dados. |

## Regras

O método identifica a fonte, não sua confiabilidade. QR/XML/imagem ainda exigem validação dos campos ambíguos; uma imagem nunca comprova pagamento nem pagador.

O arquivo original fica privado e a extração guarda a versão do processador.
