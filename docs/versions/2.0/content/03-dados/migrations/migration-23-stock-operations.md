---
id: migration-23-stock-operations
title: Migration 23 — Reposição, reservas e itens individuais
description: Evoluções do estoque para listas de compra, reservas e bens físicos identificáveis.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados, estoque
related: planejamento-2-0, migrations, modelo-de-dados, enums, estoque-despensa, migration-22-stock
source_refs:
updated: 2026-10-26
---

## Estado e dependências

Contrato proposto para uma etapa posterior à Migration 22. Nenhuma tabela obriga lançar a lista de compras, reserva ou número de série no primeiro corte. As funcionalidades dependem da base de movimentos e só entram depois que a operação agregada/lote estiver validada.

## stock_units — identificação individual

| Campo ou grupo | Contrato |
| --- | --- |
| record_id, stock_item_id, public_label | Unidade física individual, com etiqueta interna estável para consulta/QR. |
| serial_number nullable, brand_model_snapshot | Série do fabricante e identificação preservada no momento do cadastro. |
| condition, acquired_on, warranty_expires_on | StockUnitCondition, data conhecida e vencimento de garantia opcional. |
| notes, archived_at | Observação e arquivamento sem remover eventos anteriores. |

Adicionar stock_unit_id e custodian_party_id nullable a stock_movement_lines. Para StockTrackingMode Individual, cada movimento aponta a unidade exata e usa quantidade absoluta 1; o sinal define entrada ou saída. A localização e custódia atuais são derivadas dos movimentos. Número de série é privado e único dentro do item quando informado. public_label não contém segredo.

Uma unidade individual pode ficar emprestada a uma Party e depois voltar. Checkout guarda custodian_party_id e o local de saída; Return informa Party e local de entrada. Condição avariada ou descartada é revisada e não transforma automaticamente o objeto em Asset.

## stock_reservations e stock_reservation_lines

stock_reservations guarda record_id, StockReservationStatus, purpose_record_id nullable, reserved_for, expires_at e notas. O propósito pode apontar para tarefa, evento de veículo, compromisso ou outro Record do mesmo espaço. stock_reservation_lines guarda item, lote/unidade/local opcional e quantidade positiva na unidade-base.

Adicionar stock_reservation_line_id nullable a stock_movement_lines. Reservar não altera saldo físico; reduz saldo disponível. Uma reserva ativa não pode superar disponível por item/lote/local/unidade, com lock transacional. Fulfilled ocorre ao vincular o consumo real; liberação, cancelamento e expiração devolvem disponibilidade sem criar saída física. Histórico de cada transição fica auditável.

Reservations para lista de refeições, kits ou eventos são opcionais. Não reservar automaticamente só porque há uma tarefa ou compra relacionada.

## stock_restock_lists e stock_restock_list_items

stock_restock_lists guarda record_id, nome, RestockListStatus, needed_by e notas. stock_restock_list_items guarda lista, stock_item_id nullable, product_id nullable, descrição, quantidade/unidade desejada, local de destino e notas. Se não houver cadastro, descrição livre é suficiente para inserir o item e a associação pode ser feita depois.

Adicionar stock_restock_list_item_id nullable a stock_movement_lines para medir o recebimento associado à lista. Recebimento parcial permanece parcial; concluir lista não significa que tudo foi comprado ou armazenado. Lista pode nascer manualmente ou da sugestão de ponto mínimo/alvo, sempre com revisão antes de marcar necessária.

Uma lista não cria transação financeira, pedido a fornecedor nem entrada física. A compra continua opcional no módulo financeiro; só uma entrada confirmada muda o saldo.

## Índices, estados e integridade

- StockUnit pertence ao mesmo item/espaço; unique serial por item quando preenchido, public_label único no espaço.
- Tracking Individual exige uma unidade identificada nas linhas; modos Aggregate e Lot não aceitam stock_unit_id.
- Uma unidade não pode estar em dois locais nem ser emprestada duas vezes ao mesmo tempo. Concorrência é protegida pela sequência de movimentos e bloqueio por unidade.
- Quantidade de reservas ativas nunca excede disponível. Transições de reserva são atômicas e idempotentes.
- Quantidade recebida associada à linha de reposição não supera a quantidade pedida sem confirmação de ajuste.
- Listas, reservas e suas linhas obedecem escopo pessoal e podem ser exportadas/apagadas segundo as regras dos registros vinculáveis.

## Componentes previstos

RegisterStockUnit; CheckoutStockUnit; ReturnStockUnit; CreateStockReservation; FulfillStockReservation; BuildRestockSuggestions; EditRestockList; ReceiveRestockItems.

## Checklist

- [ ] Prototipar empréstimo, reserva parcial e recebimento parcial antes de fechar estados.
- [ ] Provar que saldo físico, disponível, reservado e em custódia reconciliam.
- [ ] Validar revisão/expiração de reservas sem ajuste físico implícito.
- [ ] Cobrir itens livres de lista e vínculo posterior com Product e FinancialTransactionItem.
