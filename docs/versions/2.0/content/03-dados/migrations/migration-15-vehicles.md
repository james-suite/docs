---
id: migration-15-vehicles
title: Migration 15 — Veículos, serviços e combustível
description: Contrato lógico de tabelas, FKs, índices e conversão: veículos, diário, serviços e combustível.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, veiculos, financas, patrimonio, componentes-tecnicos, enum-vehicle-event-type, enum-vehicle-financial-link-purpose
source_refs:
updated: 2026-10-26
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 02, 03, 06, 13. A tabela de bens do pacote 18 pode apontar para um veículo; o cadastro de veículo não depende da criação de um bem. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `vehicles`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, nickname, make, model, model_year` | Identificação opcionalmente detalhada; cadastro é útil mesmo sem um bem patrimonial. |
| `plate, vin, distance_unit, acquired_on, disposed_on, archived_at` | Dados privados, unidade e ciclo de posse. |

## `vehicle_odometer_readings`

| Campo ou grupo | Contrato |
| --- | --- |
| `vehicle_id, observed_at, value, unit, source` | Valor exato com unidade e procedência; pode ser salvo sem abastecimento, serviço ou transação. |
| `correction_of_id, notes` | Correção explícita para erro/troca de instrumento. |

## `vehicle_events`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, vehicle_id, event_type, occurred_on, occurred_at, time_precision` | Evento real com VehicleEventType; dia conhecido não vira instante inventado. |
| `title, details_markdown, party_id, odometer_reading_id` | Título/texto do diário, prestador ou contato e leitura opcional. |

## `vehicle_event_items`

| Campo ou grupo | Contrato |
| --- | --- |
| `event_id, position, description, quantity, unit_price` | Várias operações/peças/mão de obra no mesmo evento; precisão exata, sem copiar valores financeiros. |

## `vehicle_financial_links`

| Campo ou grupo | Contrato |
| --- | --- |
| `record_id, vehicle_id, financial_transaction_id, financial_transaction_item_id` | Liga custo/compra ao veículo; item opcional para identificar a linha fiscal exata e evitar FK de veículo no núcleo financeiro. |
| `vehicle_event_id` | Evento realizado opcional. Nulo significa que o custo foi associado ao veículo, não que um serviço aconteceu. |
| `purpose, quantity_allocated, amount_allocated_minor, notes` | Purpose distingue combustível, compra destinada ao veículo, custo de serviço e outro; valores de rateio opcionais e exatos nunca excedem a linha original. |

Links não criam transações, pagamentos ou lançamentos. Um item comprado pode primeiro ser ligado ao veículo sem evento e depois atualizado para apontar ao evento real de instalação/troca; a data de compra e a data do serviço continuam distintas. Se uma linha for dividida entre vários veículos ou serviços, quantidades/valores alocados não podem exceder o original. Relatórios contam a despesa financeira uma vez, mesmo quando há link de evento.

## `fuel_events`

| Campo ou grupo | Contrato |
| --- | --- |
| `vehicle_event_id, fuel_type, quantity, quantity_unit, full_tank` | Dados específicos de abastecimento, ligados a evento com tipo Fueling; valor não se duplica aqui. |
| `odometer_reading_id, place_id` | Leitura opcional e lugar/filial do posto; FKs adicionadas após places quando a ordem física exigir. |

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
- Todos os FKs de vehicle, transaction, item e event compartilham `space_id`; se houver `financial_transaction_item_id`, o item pertence à transação indicada. Indexar linha do tempo por veículo/data e links nos dois sentidos.
- Uma nota fiscal com um único item de combustível e quantidade conhecida permite calcular preço efetivo por litro após desconto. Com vários itens e desconto de cabeçalho, exigir atribuição revisada; não escolher arbitrariamente o combustível.
- O pagador está no pagamento financeiro, não no veículo. Pagamento por contato ou desconhecido não movimenta uma conta pessoal nem gera acerto automaticamente.
- `vehicle_events` registra fato ocorrido. Uma compra para o veículo sem `vehicle_event_id` não afirma que a peça foi instalada ou o serviço executado; `Note` guarda diário livre sem efeito financeiro automático.
- O link patrimonial é opcional e fica em `assets.vehicle_id`, único quando presente. Avaliação do bem não é custo de oficina e não cria caixa.
- Migrations físicas de fuel_events/place_id respeitam a dependência do pacote 13; manter DDL e backfill separados. Um registro de abastecimento pode sugerir leitura nova, mas não exige que ela exista.

## Conversão do James 1.x

Módulo novo. Despesas antigas podem ser relacionadas a veículo depois, sem copiar a transação.

## Componentes previstos

RecordVehicleEvent; LinkVehicleFinancialItem; ImportFiscalDocument; RecordFuelEvent; CalculateConsumption; EvaluateMaintenanceDue; guardas de duplicidade financeira e de quantidade usada.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
