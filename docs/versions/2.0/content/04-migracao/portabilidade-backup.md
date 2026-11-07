---
id: portabilidade-backup
title: Exportação completa, relatórios e backup
description: Portabilidade contínua de todos os módulos e recuperação integral da instalação multiusuário.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento, exportação
related: migracao-2-0, migration-20-imports, compatibilidade-legado, documentos-arquivos, tags, financas
source_refs:
updated: 2026-11-07
---

## Escopos e entrega

| Operação | Conteúdo | Destinatário e uso |
| --- | --- | --- |
| Exportação completa da conta | Todos os dados pessoais retidos de todos os módulos, versões e vínculos, com arquivos originais e metadados | Própria conta autenticada, portabilidade pelo navegador |
| Exportação de relatório | Resultado filtrado, medida, dimensões, regras de agregação e cobertura | Própria conta autenticada, análise em planilha e outros sistemas |
| Backup integral da instalação | Banco completo, armazenamento privado, configurações necessárias, versões e segredos de recuperação protegidos | Operador autorizado da instalação, restauração do serviço multiusuário |

A exportação continua disponível no James 2.0 ao longo do uso. A migração do 1.x utiliza o mesmo princípio de pacote versionado e adaptadores por versão. Cada módulo precisa ter um contrato de exportação antes da entrega.

## Pacote completo da conta

Gerar ZIP em fila com manifesto, JSON estruturado e originais privados. Incluir transações, itens, valores originais, ajustes globais e alocações parciais, pagamentos, contas/cartões ativos e arquivados, faturas/parcelas, categorias e hierarquia, tags/aliases/assignments, pessoas e papéis nos vínculos, acertos e suas alocações. Preservar moeda, precisão, datas, fuso e proveniência.

Incluir também produtos/preços, veículos, bens, documentos e todas as versões preservadas, diário/revisões, agenda, tarefas, lugares/check-ins e contexto geográfico ainda retido. Abranger preferências pessoais, notificações/histórico de entrega sanitizado, regras, dados de IA retidos, conversas e fontes/derivados pessoais quando existentes. Identificar arquivados e lixeira. Arquivos expirados ou já eliminados seguem indicados como indisponíveis com a justificativa conhecida.

Identidades estáveis e relações exportadas permitem reconstruir a estrutura. Separar configurações de conexão dos segredos. Senhas, API keys e tokens precisam ser configurados novamente numa importação pessoal. O manifesto enumera todos os grupos incluídos, contagens, bytes/hashes, restrições e exclusões técnicas. Dados de outros usuários mantêm seu isolamento.

Um catálogo de exportadores classifica cada tabela/coleção como dado pessoal exportável, infraestrutura, derivado reconstruível ou segredo operacional. A revisão de uma nova migration exige declarar esse tratamento. Uma tabela pessoal sem exportador impede marcar o pacote completo como concluído.

Capturar snapshot coerente de banco e versões de arquivos. Falha de integridade interrompe a finalização e disponibiliza diagnóstico. Link de download privado e temporário, reautenticação para dados sensíveis e expiração do artefato fazem parte do fluxo. Conservar o original ao lado dos derivados exportados permite reprocessamento posterior.

## Relatórios reproduzíveis

Oferecer CSV por conjunto com relações normalizadas e JSON/manifesto acompanhante. Registrar versão, instante do snapshot, filtros, medida, moeda, período, semântica de datas e política de agregação. IDs de categorias/tags/contatos e seus rótulos/caminhos permitem interpretar a planilha mesmo após renomeações futuras.

Exportar transações e itens em conjuntos separados. Pagamentos, parcelas e liquidações têm suas próprias linhas e chaves de ligação. Uma visão achatada deve declarar sua granularidade para que várias tags ou pagamentos associados preservem o total correto.

Medidas de consumo, caixa, dívida e acerto trazem nomes e bases próprios. O relatório inclui valores sem categoria, resíduos de descontos/acréscimos, cobertura de preço unitário e sobreposição de tags. Um indicador pode ser reproduzido a partir dos fatos e metadados exportados.

## Backup integral e restauração

Planejar cópia consistente de todo o banco e dos bytes de armazenamento, inclusive documentos, versões, avatares e demais coleções. Preservar versão do aplicativo, schema, dependências e configuração para reconstruir o ambiente. Cofre/chaves de criptografia são necessários para recuperar credenciais cifradas. Guardá-los com backup cifrado e acesso exclusivo do operador.

O procedimento é administrativo e recupera a instalação inteira. Antes de reabrir o serviço restaurado, validar contagens, hashes, relações, saldos, arquivos e escopo de usuários. Suspender entregas externas e automações durante a restauração, reconciliar tarefas já realizadas e invalidar sessões conforme a política de recuperação.

Definir destino, frequência, retenção, cópias independentes e exercícios periódicos de restauração antes de produção. Registrar resultados e tempo de recuperação. Um pacote pessoal é importado na própria conta, enquanto o backup integral é restaurado em ambiente operacional controlado.

## Publicação e recuperação do pacote

O manifesto mantém referência às versões originais fixadas no snapshot, com retenção temporária enquanto a geração estiver ativa. O pacote completo só recebe conclusão quando todos os grupos declarados e seus arquivos conferirem. Uma conta ainda pode receber dados novos durante exportação, e esses dados pertencem ao próximo snapshot.

O importador registra adaptadores suportados por formato e versão. A entrega de portabilidade do 2.0 inclui validação e importação de seu pacote pessoal em conta de destino vazia, além do adaptador legado 1.x. Ambos remapeiam IDs e preservam relações. Credenciais e associações com dispositivos/calendários exigem reconfiguração explícita.

Relatórios CSV escapam células que possam ser interpretadas como fórmula por planilhas e declaram o escape no manifesto. O JSON preserva valores originais, decimais como strings e unidades monetárias inteiras sem perda de precisão. Separadores, codificação e fuso ficam identificados.
