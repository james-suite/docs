---
id: migration-20-imports
title: Migration 20 — Exportação, importação e mapeamento legado
description: Contrato lógico de tabelas, FKs, índices e conversão: exportação, importação e mapeamento legado.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, migracao-2-0, documentos-arquivos, migration-22-documents, componentes-tecnicos
source_refs:
updated: 2026-11-03
---

## `import_runs`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id, space_id` | Destino fixado no servidor pela conta autenticada; imutável. |
| `source_instance_id, export_id, format_version, package_sha256` | Identidade de origem, pacote e versão verificáveis. |
| `status, manifest, validation_report, reconciliation_report` | ImportStatus, prévia e comparações. |
| `uploaded_media_id, expires_at, confirmed_at, completed_at` | Arquivo privado temporário e marcos. |
| `preview_hash, destination_revision, converter_version` | Identificam exatamente a prévia aprovada e permitem rejeitar mudanças de origem/destino. |

## `import_items`

| Campo ou grupo | Contrato |
| --- | --- |
| `run_id, source_table, source_id, status` | Item permitido por conversor; UNIQUE(run_id, source_table, source_id). |
| `normalized_payload, failure_code` | Staging validado e temporário, com acesso restrito e limpeza definida. |

## `legacy_id_maps`

| Campo ou grupo | Contrato |
| --- | --- |
| `space_id, source_instance_id, source_table, source_id, output_role, output_index` | Chave única de conversão: uma origem pode produzir documento, pagamento, alocação e parcela. Papéis/índices são determinísticos por versão do conversor. |
| `target_kind, target_id, import_run_id, source_metadata` | Destino por mapa fechado; várias origens podem apontar ao mesmo destino (transferência consolidada). Metadados seguros preservam datas e precisão relevantes. |

## `data_exports (backport separado no 1.x)`

| Campo ou grupo | Contrato |
| --- | --- |
| `requested_by_user_id, export_id, format_version, source_instance_id` | Solicitação via navegador; schema implementado posteriormente no repositório 1.x por migration nova. |
| `private_path, package_sha256, status, expires_at` | Geração em fila, download autenticado e expiração. |

## Integridade, índices e comportamento

- Fluxo confirmado: exportar arquivo completo no navegador do 1.x; entrar na própria conta do 2.0; enviar arquivo no navegador; conferir prévia; confirmar importação para essa conta.
- Senhas, tokens, API keys, sessões, permissões e proprietário do arquivo não são transportados como autoridade. Anexos e histórico de domínio entram quando suportados pelo formato, com perdas explicitamente relatadas.
- A primeira versão do importador propõe exigir destino sem dados dos domínios importados; não mesclar arbitrariamente dois financeiros preenchidos. Arquivo repetido é reconhecido; novo export da mesma origem já migrada não duplica dados nem vira sync implícito.
- Validar ZIP sem caminhos absolutos/../, symlinks ou extração ilimitada; limites comprimidos/descomprimidos e contagem declarados antes do upload. Hash assegura integridade, não autenticidade de origem.
- Validar em staging privado; publicar dados relacionais em transação após prévia aprovada e guardar arquivos previamente preparados. Falha reverte banco e limpa arquivos órfãos; reprocessamento usa mapeamentos únicos. Pacotes além do limite inicial exigem estratégia própria antes de ampliar o tamanho.
- No início da confirmação, bloquear o espaço e revalidar destino vazio (fora seeds técnicos), proprietário e revisão da prévia. UNIQUE parcial em import_runs impede duas runs running no mesmo espaço. Todas as gravações de domínio participam da guarda de importação; mudanças concorrentes causam prévia expirada ou manutenção temporária, sem apagar dados novos do usuário.
- A FK opcional spaces.active_import_run_id é adicionada depois de import_runs existir. A liberação da guarda em sucesso/falha é recuperável; não depender de um processo vivo para destravar a conta.
- Reconciliação inclui séries por mês/conta e não só saldo final. Alterações temporais causadas pela nova regra de cartões são mostradas e exigem aceite explícito.
- Não marcar completed antes de conferir saldos, dívidas, cotas, tags e documentos/versões/vínculos. Não disparar IA, recorrências ou notificações externas durante a importação.

## Conversão do James 1.x

Todas as tabelas legado são tratadas no conversor versionado e na nota compatibilidade-legado. Anexos são convertidos conforme a Migration 22; o staging mantém os bytes originais e o relatório de inconsistências até expiração. Exportação é portabilidade, não um restore direto do banco.

## Componentes previstos

RequestDataExport no 1.x; InspectImportPackage; PreviewImport; ConfirmImport; ReconcileImport; CleanupImportFiles; Policies do run, download e relatório.
