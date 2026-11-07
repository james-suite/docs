---
id: compatibilidade-legado
title: Conversão e reorganização dos dados do James 1.x
description: Preparação revisável dos dados antigos para aproveitar o modelo do James 2.0.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: migracao-2-0, financas, cartoes, acertos, tags, migration-20-imports, portabilidade-backup
source_refs:
updated: 2026-11-07
---

## Objetivo da compatibilidade

O 2.0 recebe os dados do 1.x e permite reorganizá-los para aproveitar seu modelo. O arquivo original é a referência de partida. O resultado pode reunir, dividir, corrigir, reclassificar e enriquecer registros conforme um plano de transformação revisado pela pessoa.

O schema e as regras do 2.0 são definidos pelas necessidades do produto. Conversores e preparação traduzem a origem para esse contrato. O destino pode ter outros IDs, outra quantidade de registros, outra classificação e outra organização das relações.

Manter o pacote original verificável durante preparação e conferência. Registrar a proveniência, as decisões e os efeitos relevantes das mudanças. O usuário pode preparar os dados com ferramentas externas ou um agente de IA antes de enviá-los pelo navegador.

## Fluxo de preparação

1. Exportar o pacote completo do 1.x e guardar a cópia original.
2. Gerar uma primeira conversão para o modelo-alvo do 2.0.
3. Revisar ou transformar a cópia de trabalho, com edição manual, regras determinísticas ou agente opcional.
4. Entregar o original e a preparação ao importador, com versão de formato, referências de origem e hashes.
5. Conferir a prévia do resultado, diferenças de organização, correções financeiras e relações pendentes.
6. Confirmar a versão do plano e publicar o conjunto validado na própria conta.

O importador também oferece conversão direta quando a pessoa quiser apenas receber uma proposta inicial. A preparação pode ser repetida e comparada antes da publicação. Cada revisão invalida a prévia anterior.

## Transformações permitidas

| Origem | Organização possível no 2.0 | Conferência necessária |
| --- | --- | --- |
| Contatos duplicados ou empresas cadastradas como pessoa | Unir Parties, converter tipo, separar filiais, organizar aliases e grupos | Revisar identidade, notas, papéis, participações no mesmo rateio e obrigações |
| Tags genéricas, redundantes ou protegidas no 1.x | Criar hierarquia de categorias, fundir/renomear tags específicas e substituir classificações por vínculos de domínio | Mostrar atribuições antigas e novas por transação/item, mantendo a procedência |
| Transações, parcelas e faturas | Agrupar parcelas numa compra com financiamento, separar compra de pagamentos e reconstruir relações de fatura | Conferir total, datas conhecidas, prestações, pagamentos e dívida |
| Itens resumidos ou mal classificados | Separar ou reunir linhas, corrigir quantidade/unidade e associar produtos confirmados | Demonstrar origem dos detalhes, reconciliar valores e indicar informações complementadas |
| Linhas negativas | Representar descontos/acréscimos em ajustes e atribuí-los aos itens correspondentes quando conhecido | Preservar sinal/valor original na transformação e conferir o efeito financeiro |
| Contas e cartões redundantes | Consolidar cadastros compatíveis, organizar nomes e decidir arquivamento | Reconciliar saldos, moedas, faturas, pagamentos e recorrências redirecionadas |
| Acertos e itens “Minha Parte”/“Reembolso” | Reconstruir cotas, contribuições, financiamento e obrigações do 2.0 | Rever quem consumiu, quem pagou, valores em aberto e liquidações efetivas |
| Anexos repetidos | Organizar documentos únicos com versões e vários vínculos | Confirmar equivalência, hashes, contexto e versão usada em cada evidência |
| Notificações e auditoria antigas | Manter histórico identificado e relacioná-lo aos destinos reorganizados | Preservar autoria/data disponíveis e distinguir fatos antigos das ações de preparação |

Uma transformação pode reunir várias origens em um destino ou produzir vários destinos a partir de uma origem. Registros novos podem ser complementados manualmente ou por documentos adicionais. Informações desconhecidas recebem esse estado ou passam por confirmação explícita quando forem estimativas.

As decisões sobre descarte, fusão ou substituição aparecem na prévia. Conteúdo histórico que a pessoa escolher conservar pode ser mantido em snapshots privados ou no arquivo original retido. O relatório e o mapa de transformação continuam exportáveis. Definir retenção do pacote integral e permitir baixá-lo antes de sua expiração.

## Ajuda opcional de IA

Um agente pode analisar uma cópia da exportação, identificar candidatos a duplicidade, sugerir categorias/tags, reconstruir relações e produzir o conjunto preparado. Ele pode gerar um arquivo de preparação no formato documentado do importador. A pessoa escolhe a ferramenta e autoriza quais dados ela poderá processar.

Dentro do James, uma eventual assistência de preparação terá autorização, conexão, orçamento e retenção próprios. Essa possibilidade é uma extensão separada das funções iniciais de IA. A importação permanece utilizável por conversão determinística e edição manual.

O resultado do agente é entrada de dados para o validador. O servidor verifica formatos, escopo, IDs de origem, referências, limites monetários e invariantes do 2.0. A publicação usa o grafo preparado aceito na prévia. Scripts, SQL e instruções vindos de arquivos ou respostas de IA são tratados como conteúdo não executável pelo importador.

Apresentar hipóteses e a evidência usada em cada agrupamento. Para parcelas com descrições semelhantes, por exemplo, mostrar número/quantidade de parcelas, valores, datas e fatura antes de confirmar a compra reconstruída. Uma informação sugerida pode ser aceita, corrigida ou deixada como pendência pela pessoa.

## Reconciliação de resultado transformado

Comparar três conjuntos de informação.

| Conjunto | Finalidade |
| --- | --- |
| Referência original do 1.x | Mostrar saldos, datas, classificações, faturas, acertos e contagens de partida |
| Resultado esperado após o plano aprovado | Explicar a reorganização, mudanças de representação e correções confirmadas |
| Resultado efetivamente publicado | Conferir o banco e os arquivos contra o resultado esperado |

Alterar uma classificação pode mudar gráficos sem mudar dinheiro. Unir contas exige comparar seus saldos pelo agrupamento aprovado. Corrigir um erro histórico pode mudar um saldo, desde que a diferença tenha origem, motivo e confirmação. Cada delta financeiro é revisado separadamente das mudanças apenas organizacionais.

A aceitação exige que o resultado publicado corresponda ao plano validado e que as diferenças perante a origem estejam explicadas. Um ajuste financeiro explícito pode fazer parte de uma correção escolhida. Ajustes gerados apenas para esconder falhas de conversão impedem a conclusão.

## Casos que exigem atenção

O 1.x pode ter associado a saída bancária à data da compra ao quitar uma fatura. O plano do 2.0 pode reorganizar esse efeito na data do pagamento efetivo, mostrando a diferença por período e evitando duplicidade. Datas complementadas preservam a precisão e a autoria dessa complementação.

A regra antiga de saldo dos acertos pode produzir diferenças em relação às obrigações do 2.0. A preparação permite reconstruir e corrigir esses vínculos com revisão, conservando o diagnóstico da origem. Grupos podem ser refeitos antes da importação quando a pessoa confirmar a nova distribuição.

Itens com preço negativo podem virar ajustes estruturados. Tags antigas podem ser substituídas por categorias ou relações de contato/veículo quando essa organização for aprovada. O registro de preparação conserva o caminho entre origem e destino.

Dados removidos definitivamente no 1.x podem ser complementados por arquivos ou informações fornecidos pela pessoa. Indicar a origem do complemento. Informações que permanecerem desconhecidas continuam identificadas na prévia e nos relatórios de cobertura.

## Limites de autoridade e publicação

O servidor fixa o proprietário pela conta autenticada. Cada destino é validado no mesmo espaço, com remapeamento de IDs e versões de formato conhecidas. A identidade informada no arquivo serve para proveniência.

A confirmação fixa pacote, plano, revisão, grafo-alvo e relatório financeiro. Alteração posterior da preparação exige nova validação. A publicação aplica esse resultado com atomicidade e recuperação de arquivos previstas no pacote 20. Notificações e automações operacionais começam após a migração aceita e sua configuração explícita.
