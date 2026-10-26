---
id: compatibilidade-legado
title: Mapa de compatibilidade com o James 1.x
description: Conversão por entidade e reconciliação dos efeitos financeiros e históricos.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: migracao-2-0, financas, cartoes, acertos, tags, migration-20-imports
source_refs:
updated: 2026-10-25
---

## Princípio

O pacote de exportação preserva a forma original e os vínculos. Uma linha pode produzir vários objetos no destino, com papéis de conversão identificados; várias linhas antigas podem produzir uma transferência consolidada. O importador traduz para o novo modelo e registra cada chave de origem. Não reutilizar PKs do arquivo como PKs da nova conta nem deduzir vínculos só por descrição.

## Conversão por área

| Origem | Destino proposto | Preservação e cuidado |
| --- | --- | --- |
| User | Conta autenticada e seu Space | Não restaurar login, senha, sessão, 2FA ou administração; autoria antiga vira referência histórica mapeada. |
| Contact, ContactGroup e pivô | Party(Person), PartyGroup e membros | Nome, notas, métodos de contato, grupos, lixeira e mídia. Não classificar como empresa automaticamente. |
| FinancialAccount e pix_keys | FinancialAccount e identificadores | Tipo, nome, saldo reconciliado e chaves privadas; lixeira não vira arquivamento sem regra explícita. |
| FinancialCreditCard | CreditCard e regra de ciclo | Conta pagadora passa a padrão opcional, limite e datas preservados. |
| FinancialCreditCardInvoice | Invoice, parcelas disponíveis e pagamentos legados | Fechamento/vencimento reais, amount_paid, paid_at, referências a juros e pagamento agregado. Não inventar datas de parcelas parciais perdidas. |
| FinancialTransaction | Documento e pagamentos/alocações conforme instrumento | Manter amount, date, description e estado; compra de cartão não vira automaticamente saída de banco adicional. |
| FinancialTransactionItem | Item com valores exatos, classificação e origem | Quantidade, preço, total, tags próprias e identificação da parte de terceiro quando demonstrável. |
| transfer_pair_id | Transferência única | Validar pares, direção, valor e moeda; não importar as duas linhas como duas despesas. |
| FinancialRecurrence | Regra, template e ocorrências já geradas | Datas nominais/chaves de ocorrência sem reexecução do passado. |
| FinancialTag e financial_taggables | Tags e proposta de categorias | Preservar aliases, cores/ícones, is_primary e usos por item; códigos especiais substituem IDs mágicos. |
| Settlement e SettlementGroup | Obrigações, rateios, cotas, contribuições e liquidações | Traduzir TheyOwe/IOwe/TheyPaid/IPaid, preservar datas e valores; não recalcular rateio legado pelo algoritmo novo. |
| ContactSettlementArchive | Arquivamento contextual de acertos | Não arquivar o contato inteiro quando só o saldo foi arquivado. |
| media | Anexos privados com mapa de IDs | Validar hash, tipo, tamanho, dono e entidade referenciada; registrar arquivo ausente. |
| notifications | Histórico pessoal compatível | Preservar leitura onde houver e não reenviar canais. Payload não suportado fica disponível no relatório/artefato legado. |
| activity_log | Histórico legado identificado | Preservar data/autoria disponível e sujeito mapeado; não reapresentar cada registro como ação recém-executada pelo importador. |
| cache, jobs, sessões e tokens | Não importar para execução | Infraestrutura efêmera e segredos não são dados de negócio migráveis. A omissão é explicitada no manifesto. |

## Colisões de classificação

Se o legado contiver tags distintas que colidem na nova normalização, a prévia exige decidir entre fusão revisada e nomes desambiguados preservando o label/ID original. Não perder vínculos nem fundir tags silenciosamente. Categoria derivada de tag primária usada em receitas e despesas recebe correspondências separadas por direção.

## Reconciliação financeira

Antes de importar, o exportador deve guardar contagens, saldo por conta conforme a regra atual, valor de faturas, total pago, compromissos e saldos por contato. O 2.0 recalcula os equivalentes e mostra diferença por linha/contexto, não apenas um total final que permite compensações acidentais.

O 1.x pode mover compras para a conta e marcar posted ao quitar uma fatura; o importador não pode acrescentar outro débito bancário para a mesma quitação. Pagamento parcial pode estar agregado em uma transação mutável. O conversor usa a versão do formato e a topologia de referências, não somente o status.

O saldo dos acertos antigos usa uma regra que limita acumuladores por dia. O novo modelo de obrigações pode evidenciar diferenças. Preservar os eventos antigos e mostrar um diagnóstico; não gerar ajuste silencioso para forçar igualdade.

Quando não existir detalhe suficiente para reconstruir novos conceitos, guardar granularidade legada identificada e bloquear a conversão ambígua. O relatório deve dizer o que se sabe e o que não pode ser recuperado. Não agrupar parcelas históricas só porque compartilham descrição.

## Datas e mudanças nas séries históricas

O 1.x guarda várias datas somente com precisão de dia. O 2.0 deve manter a data local efetiva e um instante opcional, distinguindo data do fato, quitação conhecida e momento de importação. Não usar meia-noite como se fosse o instante real recuperado.

No cartão, o 1.x pode afetar o banco na data de cada compra ao quitar a fatura depois. Normalizar para o pagamento real da fatura pode preservar o saldo final e mudar o caixa por mês. O manifesto guarda a série legada; a reconciliação mostra por conta/período a série antiga, a nova e o motivo de cada diferença. Não prometer equivalência temporal automática entre duas regras diferentes. A escolha de apresentação/compatibilidade precisa ser aprovada antes de concluir o conversor.

## Garantias

Uma importação concluída conserva todos os dados de negócio aceitos pelo formato, com anexos e vínculos verificados. Exceções técnicas e perdas de precisão devem aparecer antes da confirmação. Dados rejeitados não desaparecem: o pacote original e o relatório permitem corrigir o exportador ou criar um conversor melhor.
