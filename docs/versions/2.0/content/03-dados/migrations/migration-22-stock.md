---
id: migration-22-stock
title: Migration 22 — Estoque, despensa e movimentações
description: Estrutura para itens controlados, árvore de locais, lotes, saldo por movimentos e contagem física.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, estoque
related: planejamento-2-0, migrations, modelo-de-dados, enums, estoque-despensa, migration-06-transactions, migration-21-products
source_refs:
updated: 2026-10-26
---

## Estado, ordem e dependências

Contrato proposto, sem migration PHP executável. A Migration 22 depende de 02 (records), 03 (parties para fornecedor), 06 (financial transaction items, associação opcional) e 21 (products, identificadores e unidades). Criar tabelas-base e extensões de FK em arquivos separados quando a ordem física real exigir.

O estoque pertence ao espaço pessoal e não exige cadastro no módulo financeiro. As relações com finanças, produtos e fornecedores são opcionais. Consulte [Estoque e despensa](doc:estoque-despensa) para comportamento e etapas.

## stock_categories

| Campo ou grupo | Contrato |
| --- | --- |
| record_id, parent_id | Categoria pessoal hierárquica; pai do mesmo espaço ou nulo. |
| name, normalized_name, display_order | Nome, unicidade normalizada entre irmãos e ordem de apresentação. |
| icon, archived_at | Apresentação opcional e arquivamento sem retirar classificação histórica. |

Proibir ciclos e impedir arquivar categoria com itens ativos sem recategorizar ou confirmar a preservação do histórico. Não usar enum para uma árvore que a pessoa pode adaptar.

## stock_locations

| Campo ou grupo | Contrato |
| --- | --- |
| record_id, parent_id, name | Local pessoal em árvore livre, como residência/cozinha/armário/prateleira. |
| kind, environment, display_order | StockLocationKind, StorageEnvironment e ordenação. Ambiente é descrição, não telemetria. |
| notes, archived_at | Instruções privadas e arquivamento rastreável. |

Local pai e filho devem ser do mesmo espaço. Impedir ciclos na árvore. Arquivar local com saldo positivo exige transferência/contagem revisada; locais já usados não são apagados.

## stock_items

| Campo ou grupo | Contrato |
| --- | --- |
| record_id, name, product_id nullable | Item de controle pessoal; Product liga o catálogo comercial quando existe. |
| category_id, base_unit_code, tracking_mode | Categoria opcional, unidade-base do catálogo MeasurementUnit e modalidade StockTrackingMode. |
| reorder_point_quantity, target_quantity | Quantidades em unidade-base, não negativas; target não menor que o ponto mínimo quando ambos existem. |
| default_location_id, default_supplier_party_id | Sugestões para entrada/reposição, não saldo nem fornecedor obrigatório. |
| opened_use_days, notes, archived_at | Prazo opcional após abertura, instruções e arquivamento. |

Nome próprio continua obrigatório mesmo com product_id: permite distinguir uso e conservação pessoais. Alterar a unidade-base exige conversão auditada. Item arquivado permanece consultável e aceita movimentação de saída, correção ou transferência necessária ao encerramento.

## stock_lots

| Campo ou grupo | Contrato |
| --- | --- |
| record_id, stock_item_id, lot_code nullable | Lote do item; código pode não existir na embalagem. |
| manufactured_on, expires_on, best_before_on | Datas distintas e opcionais, nunca inferidas quando desconhecidas. |
| received_on, opened_on, notes | Recebimento, abertura e observação privada. |

Unicidade por espaço/item/código somente quando lot_code não for nulo. Quantidade não é armazenada no lote: o saldo do lote vem das linhas de movimento e localização. Lote usado precisa pertencer ao mesmo stock item.

## stock_movements

| Campo ou grupo | Contrato |
| --- | --- |
| record_id, movement_type, source | Evento confirmado: StockMovementType e StockMovementSource. |
| occurred_at, created_by_user_id | Data efetiva declarada e autoria autenticada. |
| reason, notes, reverses_movement_id | Motivo exigido para ajuste/perda/correção; observação e vínculo de estorno. |

Um evento publicado não é editado para corrigir quantidade; criar movimento compensatório apontando ao original. Fonte FinancialItem exige ao menos uma linha com financial_transaction_item_id. Fonte Manual, InitialCount ou Gift não exige linha financeira.

## stock_movement_lines

| Campo ou grupo | Contrato |
| --- | --- |
| movement_id, stock_item_id, stock_lot_id nullable, location_id nullable | Destino/origem do saldo; lote e local devem pertencer ao item e espaço corretos. |
| quantity_delta, base_unit_code | Decimal exato, diferente de zero, na unidade-base confirmada e copiada para preservar interpretação histórica. |
| financial_transaction_item_id nullable | Linha de compra opcional que originou esta entrada. Só permitido em movimento Receipt. |

Uma linha financeira pode originar várias linhas de estoque (pacotes, lotes ou locais distintos). A soma convertida das entradas ligadas não pode ultrapassar a quantidade comprada. Registrar a relação não cria movimento financeiro. Se a compra for mista/ambígua, exigir seleção e revisão da pessoa.

## stocktake_sessions e stocktake_lines

stocktake_sessions guarda record_id, localização ou escopo, StocktakeStatus, started_at, completed_at e notas. stocktake_lines guarda sessão, item, lote/local, saldo esperado de referência, quantidade contada, horário e anotação. Concluir a sessão cria movimentos Adjustment ligados às diferenças aprovadas; não atualizar saldos diretamente.

Movimentos posteriores ao início da contagem são considerados ao reconciliar a quantidade esperada na conclusão. Uma contagem cancelada permanece auditável sem gerar ajuste.

## Integridade, índices e comportamento

- Índices por espaço/item, espaço/local, occurred_at, lote/validade e nomes normalizados. Busca de saldo consulta a soma por item/lote/local; projeção cacheada é derivada e reconstruível.
- Quantidades são exatas na unidade-base; não usar float. Impedir diferença de família ou unidade ao vincular produto, item e linha financeira.
- Saldo não fica negativo em movimento confirmado. A ação trava a combinação item/lote/local durante validação e grava todas as linhas em uma transação.
- Transferência tem linhas de saída/entrada iguais, para o mesmo item/lote, em locais diferentes. A soma do movimento é zero; validar atomicamente e cobrir concorrência.
- Receipt aumenta saldo; Consumption, Loss e saída de Checkout diminuem; Return é entrada de retorno; Adjustment precisa de motivo e pode ter qualquer sinal.
- Itens tracking_mode Lot exigem stock_lot_id em linhas com saldo; Aggregate não inventa lote. Individual e serialização entram na Migration 23.
- financial_transaction_item_id deve pertencer ao mesmo espaço e a quantidade acumulada recebida não excede a comprada. Preço e despesa continuam sendo os da linha financeira original.
- Não apagar movimentos confirmados nem suas referências. Correção usa reversão idempotente; arquivar item/local não elimina seu histórico.
- Políticas verificam isolamento de espaço em item, lugar, produto, lote, autor, fornecedor e linha financeira.

## Conversão do James 1.x

Não há estoque estruturado presumido para importar. Não converter toda compra passada em saldo atual: histórico financeiro não prova que o produto continua guardado. A pessoa pode iniciar por uma contagem e pode escolher linhas financeiras específicas como entrada, sem inferir localização, lote ou validade.

## Componentes previstos

CreateStockItem; CreateStockMovement; TransferStock; RecordConsumption; ReconcileStocktake; StockBalanceQuery; FindItemLocation; optional FinancialItemStockLink.

## Checklist

- [ ] Revisar unidade-base, precisão, modalidade e semântica de entrada/saída.
- [ ] Gerar migrations físicas separadas para catálogos, locais, itens/lotes, ledger e sessões de contagem.
- [ ] Cobrir balanço de transferência, concorrência, quantidades parciais, expiração e ajustes.
- [ ] Verificar associação financeira opcional sem criar efeito contábil duplicado.
- [ ] Incluir locais, produtos vinculados, lotes, movimentos e autoria em auditoria, exportação e importação.
