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
updated: 2026-11-07
---

## `import_runs`

| Campo ou grupo | Contrato |
| --- | --- |
| `user_id, space_id` | Destino fixado no servidor pela conta autenticada; imutável. |
| `source_instance_id, export_id, format_version, package_sha256` | Identidade de origem, pacote e versão verificáveis. |
| `status, manifest, validation_report, reconciliation_report` | ImportStatus, prévia e comparações. |
| `uploaded_media_id, expires_at, confirmed_at, completed_at` | Arquivo privado temporário e marcos. |
| `preview_hash, destination_revision, converter_version, preparation_revision, preparation_hash, target_schema_version` | Identificam exatamente a prévia aprovada e permitem rejeitar mudanças de origem/destino. |

## `import_items`

| Campo ou grupo | Contrato |
| --- | --- |
| `run_id, source_table, source_id, status` | Item permitido por conversor; UNIQUE(run_id, source_table, source_id). |
| `source_payload, source_payload_version, failure_code` | Snapshot de origem validado em staging privado. O resultado destinado ao 2.0 fica em import_prepared_entities. |

## `legacy_id_maps`

| Campo ou grupo | Contrato |
| --- | --- |
| `space_id, source_instance_id, source_table, source_id, output_role, output_index` | Chave única de conversão: uma origem pode produzir documento, pagamento, alocação e parcela. Papéis/índices são estáveis dentro do plano aprovado e versão do conversor. O mapeamento admite fusões, divisões e relações N:N com suas origens identificadas. |
| `target_kind, target_id, import_run_id, source_metadata` | Destino por mapa fechado; várias origens podem apontar ao mesmo destino (transferência consolidada). Metadados seguros preservam datas e precisão relevantes. |

## `data_exports` no 1.x e 2.0

| Campo ou grupo | Contrato |
| --- | --- |
| `requested_by_user_id, space_id, export_id, format_version, source_instance_id` | Solicitação da própria conta via navegador. Contrato adaptado à identidade do 1.x no backport e ao Space no 2.0. |
| `private_path, package_sha256, status, expires_at` | ExportStatus, geração em fila, download autenticado e expiração. |
| `scope, snapshot_at, manifest, completed_at` | Pacote pessoal completo ou relatório filtrado, snapshot consistente, cobertura e conclusão verificada. Filtros são metadados autorizados do escopo de relatório. |

## Integridade, índices e comportamento

- Fluxo confirmado: exportar arquivo completo no navegador do 1.x; entrar na própria conta do 2.0; enviar arquivo no navegador; conferir prévia; confirmar importação para essa conta.
- Senhas, tokens, API keys, sessões, permissões e proprietário do arquivo não são transportados como autoridade. Anexos e histórico de domínio entram quando suportados pelo formato, com perdas explicitamente relatadas.
- A primeira versão do importador propõe exigir destino sem dados dos domínios importados; não mesclar arbitrariamente dois financeiros preenchidos. Arquivo repetido é reconhecido; novo export da mesma origem já migrada não duplica dados nem vira sync implícito.
- Validar ZIP sem caminhos absolutos/../, symlinks ou extração ilimitada; limites comprimidos/descomprimidos e contagem declarados antes do upload. Hash assegura integridade, não autenticidade de origem.
- Validar em staging privado; publicar dados relacionais em transação após prévia aprovada e guardar arquivos previamente preparados. Falha reverte banco e limpa arquivos órfãos; reprocessamento usa mapeamentos únicos. Pacotes além do limite inicial exigem estratégia própria antes de ampliar o tamanho.
- No início da confirmação, bloquear o espaço e revalidar destino vazio (fora seeds técnicos), proprietário e revisão da prévia. UNIQUE parcial em import_runs impede duas runs running no mesmo espaço. Todas as gravações de domínio participam da guarda de importação; mudanças concorrentes causam prévia expirada ou manutenção temporária, sem apagar dados novos do usuário.
- A FK opcional spaces.active_import_run_id é adicionada depois de import_runs existir. A liberação da guarda em sucesso/falha é recuperável; não depender de um processo vivo para destravar a conta.
- Reconciliação inclui séries por mês/conta e não só saldo final. Alterações temporais causadas pela nova regra de cartões são mostradas e exigem aceite explícito.
- Não marcar completed antes de conferir saldos, dívidas, cotas, tags e documentos/versões/vínculos. A publicação executa somente o grafo preparado aprovado, com recorrências e notificações operacionais desativadas. Assistência de IA pode ser autorizada na preparação, com registro próprio da transformação.

## Exportação contínua no 2.0

Manter data_exports também no 2.0, com user_id/space_id resolvidos no servidor, export_id, format_version, snapshot_at, scope, filtros permitidos, manifest, status, private_path, package_sha256, completed_at e expires_at. scope distingue pacote pessoal completo e relatório filtrado. A entidade registra falhas e contagens sanitizadas. O mesmo usuário pode consultar andamento e baixar o resultado autorizado.

O catálogo de exportadores cobre todos os módulos e suas relações, incluindo arquivados, lixeira e versões ainda retidas. A especificação em [Portabilidade e backup](doc:portabilidade-backup) define a fronteira entre pacote pessoal, relatório e recuperação administrativa da instalação.

## Lease, atomicidade e preservação de arquivos

import_runs guarda lease_expires_at, heartbeat_at e lease_generation. O worker renova a concessão, e cada tentativa de publicação compara a geração vigente sob bloqueio do Space. Um worker antigo perde autoridade após recuperação da operação. A rotina de recuperação pode retomar a mesma run ou registrar falha e liberar a guarda de modo transacional.

As etapas de parser, validação e preparação de arquivos podem usar lotes. A publicação financeira inicial ocorre numa transação de domínio única dentro dos limites de tamanho aceitos. Falha durante essa publicação restaura o destino anterior. Repetir a confirmação pode reutilizar staging válido. Aumento de volume que exija publicação distribuída por lotes requer outro protocolo e novo gate.

Durante exportação, fixar as versões e hashes do snapshot e manter referências de retenção temporária aos bytes. A limpeza respeita essas referências até conclusão/falha/expiração recuperável da exportação. Alteração ou purge concorrente interrompe a finalização quando faltar um original obrigatório.

Arquivos de migração e XML de saúde pertencem a staging técnico privado com expiração, separado dos documentos pessoais da biblioteca. Ao importar pacote pessoal, desativar chaves de feed, assinaturas push e conexões externas até configuração explícita na conta de destino.

## Expiração e módulos suportados

uploaded_media_id do ImportRun e referências técnicas equivalentes podem ficar nulos após limpeza autorizada do staging. O registro conserva hash, versão, contagens e resultado. DataExport expirado conserva manifesto e estado, com private_path eliminado. A limpeza de arquivo pessoal continua sujeita ao contrato de documentos e evidências.

Um pacote completo informa quais módulos/schema precisa no destino. A prévia bloqueia a publicação quando faltar adaptador ou módulo necessário. O fluxo pode orientar habilitação ou atualização do destino. Importação parcial, se vier a ser aprovada, terá escopo e resultado próprios, com exclusões declaradas.

## Preparação transformável e proveniência

import_prepared_entities guarda run_id, preparation_revision, target_key, target_kind, payload_schema_version, normalized_payload, validated_at e validation_errors sanitizados. UNIQUE(run_id, preparation_revision, target_key). target_key é identidade de staging e será remapeada para ID local na publicação. Relações-alvo também usam essas chaves.

import_transformations guarda run_id, preparation_revision, operation_key, operation_type, reason, proposed_by, tool_metadata, decision via SuggestionStatus e reviewed_by_user_id/reviewed_at quando confirmada. operation_type identifica conversão, fusão, divisão, reclassificação, complemento, correção ou exclusão deliberada. Metadados de ferramenta/agente são limitados e privados.

Tabelas filhas ligam cada transformação aos import_items de origem e às entidades preparadas de destino, com chaves únicas por par. Uma correção manual pode acrescentar documento/fonte complementar. A decisão aprovada conserva diff e efeito financeiro. Hipóteses aceitas mantêm autoria e nível de certeza declarado.

import_items mantém a referência à origem. O grafo preparado define o resultado pretendido. legacy_id_maps registra os IDs efetivamente publicados para os papéis do plano, permitindo consultar todas as origens de um destino e todos os destinos de uma origem. Origens deliberadamente substituídas ou descartadas conservam a decisão no histórico de preparação.

Reconciliation_report compara baseline legado, expected_target e actual_target. Incluir deltas aprovados por moeda, conta/grupo de contas, cartão, período e contraparte. Comparação financeira final exige equivalência ao target aprovado, com alterações perante o legado justificadas. Preview_hash inclui as transformações, o grafo final e as decisões.

Modificar uma entidade preparada invalida sua validação e as dependências afetadas. Atualizar preparation_revision/hash e exigir nova prévia. Retomar falha de publicação reutiliza somente o mesmo plano aprovado. Uma revisão nova é uma operação de preparação distinta, sujeita à conferência antes de publicar.

## Aprovação do plano e entidades novas

Entidades novas do 2.0 podem ser acrescentadas ao grafo preparado, com origem manual, fonte complementar ou hipótese confirmada. Validar o schema e o módulo de destino, mesmo quando a entidade tiver zero correspondências diretas no 1.x. A operação de preparação conserva a autoria desse acréscimo.

Valores reviewed_by_user_id, reviewed_at, decision e space_id recebidos de um arquivo são metadados de origem. A autoridade de aceite é registrada pela sessão autenticada no servidor, vinculada ao hash da prévia.

A interface permite revisar transformações por grupo e confirmar o plano completo em lote, mostrando resumos e exceções financeiras. Uma revisão nova reabre o aceite dos grupos afetados. Propostas rejeitadas continuam identificadas no histórico da preparação.

## Conversão do James 1.x

Todas as tabelas legado são tratadas no conversor versionado e na nota compatibilidade-legado. Anexos são convertidos conforme a Migration 22; o staging mantém os bytes originais e o relatório de inconsistências até expiração. Exportação é portabilidade, não um restore direto do banco.

## Componentes previstos

RequestDataExport no 1.x; InspectImportPackage; PreviewImport; ConfirmImport; ReconcileImport; CleanupImportFiles; Policies do run, download e relatório.
