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
updated: 2026-10-26
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `QrCode` | `qr_code` | Leitura de chave ou código de acesso da nota. |
| `XmlUpload` | `xml_upload` | Arquivo fiscal estruturado fornecido pelo usuário. |
| `ImageScan` | `image_scan` | Foto ou imagem processada por OCR. |
| `ManualEntry` | `manual_entry` | Registro digitado sem arquivo de origem. |
| `LegacyImport` | `legacy_import` | Documento preservado por importação de dados. |

## Regras

O método identifica a fonte, não sua confiabilidade. QR/XML/imagem ainda exigem validação dos campos ambíguos; uma imagem nunca comprova pagamento nem pagador.

## Persistência

Aplicar em `fiscal_documents` conforme [Migration 06](doc:migration-06-transactions). O arquivo original fica privado e a extração deve guardar a versão do processador.

## Checklist

- [ ] Revisar as origens no fluxo de [finanças](doc:financas).
- [ ] Criar enum e rótulos no Model correspondente.
- [ ] Validar duplicidade, integridade do arquivo e revisão humana.
