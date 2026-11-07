---
id: migration-15-vehicles
title: Migration 15 — Veículos, serviços e combustível
description: Contrato lógico de tabelas, FKs, índices e conversão: veículos, diário, serviços e combustível.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, veiculos, documentos-arquivos, financas, patrimonio, migration-22-documents, componentes-tecnicos, enum-vehicle-event-type, enum-vehicle-financial-link-purpose
source_refs: https://brasilapi.com.br/docs
updated: 2026-11-07
---

## Relação com patrimônio

A tabela de bens do pacote 18 pode apontar para um veículo; o cadastro de veículo não depende da criação de um bem.

## `vehicles`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, nickname, make, model, model_year, fipe_code` | Identificação opcionalmente detalhada; `fipe_code` pode ligar o veículo a uma variante consultada e confirmada; cadastro é útil mesmo sem um bem patrimonial. |
| `plate, vin, distance_unit, acquired_on, disposed_on, archived_at` | Dados privados, unidade e ciclo de posse. |

## `vehicle_odometer_readings`

| Campo ou grupo | Contrato |
| --- | --- |
| `vehicle_id, observed_on, observed_at, time_precision, observed_timezone, value, unit, source` | Valor exato com unidade e procedência; pode ser salvo sem abastecimento, serviço ou transação. |
| `correction_of_id, notes` | Correção explícita para erro/troca de instrumento. |

## `vehicle_events`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, vehicle_id, event_type, occurred_on, occurred_at, time_precision` | Evento real com VehicleEventType; dia conhecido não vira instante inventado. |
| `title, details_markdown, party_id, odometer_reading_id` | Título/texto do diário, prestador ou contato e leitura opcional. |

## `vehicle_event_items`

| Campo ou grupo | Contrato |
| --- | --- |
| `event_id, position, description, quantity, quantity_unit_label` | Várias operações, peças ou mão de obra realizadas no evento; quantidade e unidade são opcionais e descritivas. Não guardar preço nem total aqui: custos financeiros vêm dos vínculos com transações/itens. |

## `vehicle_financial_links`

| Campo ou grupo | Contrato |
| --- | --- |
| `vehicle_id, financial_transaction_id, financial_transaction_item_id` | Liga a transação inteira ou um item ao veículo; `financial_transaction_id` sempre presente e o item opcional identifica uma linha de compra mista. Não adicionar FK de veículo no núcleo financeiro. |
| `vehicle_event_id` | Evento realizado opcional. Nulo significa que o custo foi associado ao veículo, não que um serviço aconteceu. |
| `purpose, quantity_allocated, amount_allocated_minor, notes` | Purpose distingue combustível, compra destinada ao veículo, custo de serviço e outro; valores de rateio opcionais e exatos nunca excedem a linha original. |

Links não criam transações, pagamentos ou lançamentos. Um link de transação representa o custo inteiro associado ao veículo, inclusive quando a transação não tem itens. Em compra mista, usar links por item para associar somente as linhas relevantes; não combinar um link da transação inteira com links de seus itens sem alocação explícita que impeça duplicidade. A UI pode exibir o veículo como chip/filtro parecido com uma tag, mas o vínculo continua tipado e não ocupa o catálogo de tags. Um item comprado pode primeiro ser ligado ao veículo sem evento e depois atualizado para apontar ao evento real de instalação/troca; a data de compra e a data do serviço continuam distintas. Se uma linha for dividida entre vários veículos ou serviços, quantidades/valores alocados não podem exceder o original. Relatórios contam a despesa financeira uma vez, mesmo quando há link de evento.

## `fuel_events`

| Campo ou grupo | Contrato |
| --- | --- |
| `vehicle_event_id, fuel_type, quantity, quantity_unit, full_tank` | Dados específicos de abastecimento, ligados a evento com tipo Fueling; valor não se duplica aqui. |
| `place_id` | Lugar/filial do posto; FK adicionada após places quando a ordem física exigir. A leitura do odômetro pertence ao evento-pai e não é repetida nesta tabela. |

Preço líquido e preço por litro são calculados a partir do item fiscal ligado com purpose `fuel`; não guardar uma cópia que possa divergir do financeiro. Sem comprovante, o preço pode ficar desconhecido.

## `vehicle_maintenance_plans`

| Campo ou grupo | Contrato |
| --- | --- |
| `vehicle_id, name, interval_days, interval_distance` | Prazo por tempo e/ou distância; pelo menos um intervalo positivo. |
| `last_completed_event_id, reference_reading_id` | Base real de manutenção; nunca a mera notificação. |

## Integridade, índices e comportamento

- Nenhum cálculo de consumo com float ou intervalo incompleto rotulado como média confiável.
- Oferecer ação de registrar odômetro fora do formulário de abastecimento; aceitar leituras manuais e datadas sem evento financeiro.
- Permitir correção de odômetro com justificativa; regras dependem da unidade e sequência temporal, não só de MAX(valor).
- `vehicle_financial_links` é a única ponte entre módulos; não gerar segunda despesa ao consultar veículo e finanças.
- `fuel_events` só pode existir para um `vehicle_event` com tipo `Fueling`; um evento de diário ou serviço não recebe dados de abastecimento por engano.
- `vehicle_event_items` descreve o que foi feito, não é uma segunda fonte de preço. Valor pago ou alocado é consultado pela `vehicle_financial_links`; um item comprado antes pode ser associado ao evento de instalação sem trocar sua data de compra.
- Um vínculo ao nível de transação e vínculos ao nível dos itens da mesma transação são mutuamente exclusivos, salvo rateio explícito com valores/quantidades que reconcilie exatamente a origem. Usar índices únicos adequados para impedir duplicatas no nível escolhido.
- Todos os FKs de vehicle, transaction, item e event compartilham `space_id`; se houver `financial_transaction_item_id`, o item pertence à transação indicada. Indexar linha do tempo por veículo/data e links nos dois sentidos.
- Uma nota fiscal com um único item total, combustível de quantidade conhecida, permite calcular preço efetivo por litro após desconto global. Com vários itens e desconto de cabeçalho, exigir atribuição revisada; não escolher arbitrariamente o combustível.
- O pagador está no pagamento financeiro, não no veículo. Pagamento por contato ou desconhecido não movimenta uma conta pessoal nem gera acerto automaticamente.
- `vehicle_events` registra fato ocorrido. Uma compra para o veículo sem `vehicle_event_id` não afirma que a peça foi instalada ou o serviço executado; `Note` guarda diário livre sem efeito financeiro automático.
- O link patrimonial é opcional e fica em `assets.vehicle_id`, único quando presente. Avaliação do bem não é custo de oficina e não cria caixa.
- Migrations físicas de fuel_events/place_id respeitam a dependência do pacote 13; manter DDL e backfill separados. Um registro de abastecimento pode sugerir leitura nova, mas não exige que ela exista.

## Identidade e leituras históricas

FuelEvent é uma extensão 1:1 de VehicleEvent de tipo Fueling, com UNIQUE(vehicle_event_id). Seus vínculos, anexos e tags usam o Record do evento-pai. VehicleFinancialLink é uma associação com id próprio, sem outro registro em records.

Leituras guardam observed_on, observed_at nullable, time_precision e observed_timezone. Uma leitura conhecida só por dia mantém essa precisão. Troca ou reset do odômetro abre um segmento de medição identificado. Distâncias entre segmentos exigem correspondência explícita ou ficam indisponíveis.

Validar soma das quantidades/valores dos vínculos por origem sob bloqueio. Ao corrigir desconto, quantidade ou moeda na origem financeira, revalidar as alocações do veículo e a cobertura de preço. Eventos físicos e leituras continuam sendo os fatos de execução. Um abastecimento repartido entre veículos identifica a quantidade destinada a cada evento.

full_tank aceita null para condição desconhecida. O cálculo de consumo seleciona marcos confiáveis de tanque cheio e usa abastecimentos intermediários com volume compatível. Expor as lacunas que impedem a média.

## Consistência entre pais e alocações

Leitura e evento ligados precisam pertencer ao mesmo veículo. O evento de um VehicleFinancialLink pertence ao veículo daquele vínculo. Item financeiro pertence à transação indicada, além de compartilhar o espaço. Quando aplicável, usar chave composta que inclui transaction_id para essa FK.

Uma revisão de linha financeira verifica todos os rateios de valor e quantidade dos veículos antes de publicar. O custo conhecido por litro segue o líquido e a quantidade efetivamente alocados ao abastecimento, com a cobertura dos ajustes globais identificada.

## Conversão do James 1.x

Módulo novo. Despesas antigas podem ser relacionadas a veículo depois, sem copiar a transação.

## Componentes previstos

RecordVehicleEvent; LinkVehicleFinancialItem; ImportFiscalDocument; RecordFuelEvent; CalculateConsumption; EvaluateMaintenanceDue; guardas de duplicidade financeira e de quantidade usada.
