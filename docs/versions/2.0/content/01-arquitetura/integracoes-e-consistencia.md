---
id: integracoes-e-consistencia
title: Integrações e consistência entre módulos
description: Donos dos fatos, revisões, correções, projeções e recuperação de operações entre módulos.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento, integração
related: arquitetura-2-0, modelo-de-dados, migrations, cenarios-de-validacao, financas, acertos, documentos-arquivos, notificacoes, portabilidade-backup
source_refs:
updated: 2026-11-07
---

## Dono de cada fato

| Fato | Fonte canônica | Integrações que consultam ou derivam |
| --- | --- | --- |
| Compra, valores das linhas e ajustes | Transação, itens e alocações de ajustes | Veículos, produtos/preços, acertos e relatórios |
| Origem/destino e efetivação do dinheiro | FinancialPayment e suas alocações | Conta, cartão, acerto e fluxo de caixa |
| Financiamento no crédito | CardCharge, parcelas e faturas | Compra original, agenda e notificações |
| Consumo e adiantamento por pessoa | Cotas, contribuições e financiamento das cotas | Obrigações, ficha da Party e consumo pessoal |
| Serviço realizado e odômetro | Evento/leitura do veículo | Manutenção, patrimônio e tarefas |
| Documento original | Versão publicada e seus bytes | Evidência fiscal, anexos, OCR e exportação |
| Compromisso planejado | Agenda e série | Lembretes, feed e conectores aprovados |
| Ação de usuário e sua posição | Auditoria e contexto da submissão | Histórico/mapa e rastreio da operação |
| Amostra de saúde | Observação/sessão e proveniência | Gráficos de saúde e diário relacionado |

Um vínculo genérico dá contexto e navegação. Relações com efeito financeiro, consumo, versão de evidência ou manutenção usam o contrato específico do domínio. A interface pode apresentar ambos como chips tipados, preservando sua semântica e fonte.

## Operação completa e concorrência

Cada comando com efeitos em vários registros define responsável, space_id, chave idempotente, hash de entrada e revisões esperadas. Repetir chave e conteúdo devolve o resultado anterior. Repetir a chave com outro conteúdo exige correção explícita. A durabilidade da chave acompanha o efeito de negócio, inclusive após expirar caches.

Revalidar autorização, revisão, arquivamento e dependências dentro da transação. Bloquear registros numa ordem determinística, incluindo as linhas cuja soma impõe limite. Um retry após deadlock recarrega o estado e repete a operação completa. O rascunho de contato recebe mapeamento para o Party criado pela mesma operação.

Efeitos locais que exigem atomicidade compartilham transação, como liquidação de acerto e movimento financeiro. Trabalho derivado em fila usa intenção durável e identificador da operação, com tentativas, lease e reconciliação. Reutilizar o padrão de outbox onde houver risco real de perder a intenção após commit. A publicação de arquivos e chamadas externas possuem confirmação própria e estado recuperável.

Jobs carregam IDs e espaço persistidos. Ao executar, verificam se a origem ainda existe e se a revisão e a política continuam válidas. Reprocessar a mesma revisão é idempotente. Alterar ou excluir uma fonte invalida resultados antigos antes de sua publicação.

## Propagação de mudanças

| Mudança | Efeito e revisão necessária |
| --- | --- |
| Redistribuir desconto global mantendo o total | Recalcular preço analítico e validar limites dos vínculos de veículo. Pagamentos conservam valores. Publicar nova observação de preço e invalidar comparações antigas. |
| Alterar valor total, moeda ou quantidade já financiados | Apresentar pagamentos, faturas, rateios, preços e vínculos afetados. Exigir plano de ajuste e aplicar a revisão consistente, preservando lançamentos efetivados. |
| Receber devolução de compra compartilhada | Relacionar compra e devolução, atribuir itens/cotas afetados e conferir valores já liquidados antes de atualizar obrigações. |
| Reverter liquidação | Reabrir as alocações de obrigações corrigidas e reverter o pagamento financeiro vinculado na mesma operação. |
| Adicionar versão de documento | Atualizar a biblioteca e manter evidências históricas fixadas nas versões usadas. |
| Corrigir produto, unidade ou identidade de loja | Encerrar observações substituídas, publicar correção rastreável e marcar comparações afetadas. Dados fiscais originais permanecem consultáveis. |
| Concluir serviço | Atualizar evento, tarefa vinculada e base da manutenção somente no fluxo explícito de execução real. |
| Mudar vencimento ou cancelar evento | Replanejar lembretes pela nova revisão e publicar atualização/cancelamento com identidade estável no feed. |
| Fundir Party, produto, tag ou categoria | Examinar colisões no destino e relações afetadas antes de redirecionar referências. Preservar mapa, snapshots e histórico. |
| Alterar permissão de IA ou excluir fonte | Impedir uso imediato por filtro de autorização e invalidar índices, caches e propostas pendentes. |

## Histórico e referências

Cada tabela vinculável mantém id simples e record_id único. O Model tipado é dono do conteúdo e do ciclo de vida, e a entrada em records é atualizada atomicamente como índice de identidade. RecordResolver usa mapa de tipos fechado e detecta divergência. Associações puras e extensões 1:1 usam a identidade do pai quando apropriado, sem criar outro Record para o mesmo fato.

Arquivamento preserva valores e relações históricos, e novos seletores aplicam o estado do domínio. Lixeira remove a fonte das consultas correntes e aciona invalidação. Exclusão física percorre referências e a política de retenção aprovada. A decisão de exclusão definitiva da conta continua pendente, incluindo o tratamento de auditoria funcional.

Moeda padrão e fuso do espaço têm uma fonte em spaces. Fatos preservam moeda, precisão temporal, fuso observado e dados originais relevantes. Alterar preferência afeta apresentação e padrões novos. Conversões de unidade usam catálogo e versão reconhecidos.

Localização pertence à submissão humana que a capturou. Operações automáticas podem referenciar a causa pelo identificador de correlação, mas seus eventos continuam identificados como sistema. A posição é exibida na ação humana de origem e tem retenção própria.

## Consultas, exportação e segurança de entrada

Relatórios partem do fato na granularidade declarada e aplicam relações com deduplicação antes de somar. Especificar consumo, caixa, financiamento, obrigação ou preço, além de período, moeda, cobertura e revisão. Rateios só pelo total deixam a distribuição de consumo por item/categoria com cobertura pendente.

Snapshots de exportação fixam linhas e versões de arquivos. Expiração/purge respeita referências mantidas pelo exportador até ele finalizar ou falhar. Exportação inclui relações e revisões necessárias para repetir o cálculo.

URLs externas passam por adaptador com destinos autorizados, limite de redirects, tempo, resposta e tamanho. QR de nota, payload de IA e links importados são entradas a validar. Bloquear acesso a destinos internos/metadados da infraestrutura e revalidar cada redirecionamento. Provedores de IA usam endpoint configurado e autorizado pela instalação.

Arquivos XML são processados com entidades externas e DTD desabilitadas, sem acesso de rede pelo parser. Aplicar limites de profundidade, expansão, volume e tempo. ZIPs respeitam o contrato de staging. Pré-visualizações e Markdown usam sanitização própria.

## Gates de implementação

Os cenários de compra mista, desconto parcial, rateio parcialmente financiado, estorno de liquidação, documento versionado, falha de publicação e exportação simultânea precisam ser resolvidos no protótipo antes do schema definitivo. O catálogo de pacotes separa dependências obrigatórias e FKs de módulos opcionais. Cada módulo entrega seu contrato de exportação e seus casos de falha.
