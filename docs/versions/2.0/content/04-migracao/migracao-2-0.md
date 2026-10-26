---
id: migracao-2-0
title: Migração pelo navegador para a própria conta
description: Arquivo completo exportado no 1.x e importado no navegador por sua conta no 2.0.
type: planning
diagram: migration-flow
status: proposed
visibility: public
tags: james/2.0, planejamento
related: compatibilidade-legado, migration-20-imports, financas, cartoes, enums, fases-2-0
source_refs:
updated: 2026-10-25
---

## Fluxo confirmado pelo usuário

1. No James atual, abrir Exportar meus dados, solicitar um arquivo completo e baixar pelo navegador quando a geração terminar.
2. Criar/entrar na própria conta do James 2.0 e abrir Importar dados do James 1.x.
3. Enviar o arquivo pelo navegador. A validação acontece no servidor em fila, com progresso por etapas.
4. Conferir a prévia: contagens, anexos, dados convertidos, inconsistências e comparação financeira.
5. Confirmar. Os dados vão para o espaço pessoal da conta autenticada no 2.0.
6. Acompanhar o resultado e baixar o relatório de conferência.

O arquivo nunca escolhe o dono do destino. Não há seletor de usuário nem restauração de senhas/sessões. O ID antigo serve apenas para remapear referências e autoria histórica.

{{diagram:migration-flow}}

## Pacote proposto

ZIP versionado com manifesto, arquivos JSON de dados e anexos privados. O manifesto contém `format_version`, versão da aplicação de origem, `source_instance_id` estável, `export_id`, data de geração, contagens e hashes. Dados incluem IDs de origem e todas as relações necessárias. Formato de exportação tem versão independente da versão do aplicativo.

Exportar dados de negócio, lixeira quando suportada, histórico/auditoria compatível e arquivos originais. Configurações úteis podem ser incluídas com lista permitida. Senhas, tokens, API keys, sessões, jobs e credenciais de transporte ficam fora; o manifesto informa essas exclusões. “Todos os dados” significa portabilidade dos dados de negócio, não cópia de segredos ou execução do ambiente antigo.

O exportador precisa capturar um snapshot consistente entre tabelas. Gerar em fila, com transação/snapshot e estratégia de arquivos estáveis; não montar um ZIP com relações de horários diferentes durante gravações simultâneas. Hash valida integridade, não prova autenticidade.

## Upload e prévia

Limites comprimidos/descomprimidos e de anexos devem aparecer antes de selecionar o arquivo. Validar versão, tipos, referências, hashes e tamanho; rejeitar extração com path traversal, symlink ou expansão ilimitada. Arquivo e staging ficam privados, vinculados à conta e com expiração.

A prévia não grava entidades definitivas nem chama integrações. Ela mostra exatamente os dados que faltam ou exigem revisão. Cada conversor conhece uma versão do formato; formato desconhecido é recusado com explicação, sem tentar adivinhar schema.

## Confirmação, retomada e duplicatas

Revalidar a conta, o espaço, o hash e a revisão da análise ao confirmar. Na primeira entrega, propor importação em domínios de destino ainda vazios; se já houver financeiro preenchido, explicar o bloqueio. Mesclar dois históricos completos não será uma funcionalidade implícita.

Usar o pacote e o mapa estável de origem para reconhecer reenvios. Repetir o mesmo arquivo não duplica registros; outro export da mesma instalação já migrada não inicia uma sincronização silenciosa. Uma evolução incremental exige regras próprias de conflito.

Dados são preparados em staging; gravação relacional é atômica dentro do limite de pacote suportado. Arquivos são preparados em armazenamento privado e ligados aos registros durante a publicação; falha reverte o banco e agenda limpeza de órfãos. Não expor meia importação como histórico concluído.

## Concorrência no destino

A prévia pode expirar: o usuário pode ter criado dados depois da análise. Ao confirmar, bloquear o espaço, revalidar o conteúdo de destino e impedir outra importação simultânea. As Actions normais respeitam a guarda de importação; se houver mudança, exigir nova análise. Seeds técnicos não tornam a conta preenchida, mas dados pessoais nunca são apagados para satisfazer a regra de vazio.

## Conferência obrigatória

Comparar saldo por conta e séries mensais, débito de cartão, faturas/pagamentos, valores de acertos, transações e itens, tags/categorias e contagem/hash de anexos. Diferenças devem ser localizadas no relatório. Não completar saldo com ajuste automático para esconder uma conversão incorreta.

Pagamentos agregados e datas ausentes são preservados com sua precisão real. A [compatibilidade com o legado](doc:compatibilidade-legado) explica os casos de cartão e acertos que exigem conversores próprios.

## Efeitos externos

Importação não manda e-mail/Telegram, não chama IA e não materializa recorrências. Ativar automações só após reconciliação e configuração dos canais na nova conta. O James atual permanece disponível até o aceite; mudanças posteriores no antigo não são sincronizadas automaticamente.

## Componentes e persistência

ExportWizard no 1.x; ImportWizard no 2.0; upload privado com progresso, estado de fila, relatório e botão de confirmação. Não processar todo o banco dentro de uma requisição HTTP longa. Contratos das tabelas em [Migration 20](doc:migration-20-imports), com [enums de importação](doc:enum-import-status).
