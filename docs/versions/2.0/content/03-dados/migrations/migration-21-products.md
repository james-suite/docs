---
id: migration-21-products
title: Migration 21 — Catálogo de produtos e preços observados
description: Contrato de persistência para produtos identificáveis, preços comparáveis e cestas pessoais.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, produtos-precos, financas, migration-03-parties, migration-06-transactions, migration-13-places, enum-price-observation-source
source_refs:
updated: 2026-10-26
---

## Estado, ordem e dependências

Contrato proposto, sem migration PHP executável. O número identifica uma etapa lógica, não um arquivo timestampado. Dependências: 02 (registros), 03 (partes), 06 (transações e itens) e 13 (lugares e filiais).

Adicionar as FKs de produto ao item financeiro e ao catálogo de preços após as tabelas-base. A FK do emitente fiscal para places é parte da extensão do pacote 13. Valores de notas continuam sendo fonte do fato financeiro; catálogo e observações são relações e projeções rastreáveis.

## Tabela measurement_units

| Campo ou grupo | Contrato |
| --- | --- |
| code, label, unit_family, base_unit_code | Chave estável e família física; tabela de catálogo do sistema, somente leitura por espaço. |
| conversion_numerator, conversion_denominator | Fator racional positivo da unidade para a base da família. |

Registrar códigos estáveis, família física, unidade-base e fator racional de conversão. Seeds do sistema incluem ao menos massa (g/kg), volume (mL/L) e contagem (unidade); não permitir conversão entre famílias. Numerador/denominador inteiros evitam arredondamentos intermediários. Não oferecer unidades criadas livremente enquanto conversão e comparação não tiverem semântica validada.

## Tabela products

| Campo ou grupo | Contrato |
| --- | --- |
| record_id, name, brand, variant | Produto privado do espaço; marca e variante opcionais. |
| package_quantity, package_unit_code, base_unit_code | Conteúdo da embalagem e unidade comparável; decimal exato e unidade de catálogo controlado, sem float. |
| archived_at, notes | Arquivar sem apagar compras e observações passadas. |

## Tabela product_identifiers

| Campo ou grupo | Contrato |
| --- | --- |
| product_id, scheme, country, value_normalized | Identificador como GTIN/EAN/UPC ou código local, com esquema e país. |
| source, verified_at | Origem e estado de conferência do código. |

Unicidade proposta por espaço, esquema, país e código normalizado. Um código pode sugerir produto existente; conflito ou embalagem diferente precisa de revisão. Não adotar catálogo global de produtos no primeiro corte.

## Tabela product_aliases

Guardar product_id, normalized_description, party_id nullable, source e verified_at. Unicidade por espaço/produto/emitente/descrição normalizada. party_id limita o alias a uma loja/emitente específico; a linha fiscal original continua preservada. Aliases só sugerem correspondência, não mesclam variantes.

## financial_transaction_items

Adicionar product_id nullable e normalized_base_quantity/normalized_base_unit_code quando a linha puder ser comparada. A FK composta garante espaço igual. A descrição e os números fiscais originais continuam imutáveis como snapshot da compra, mesmo se o nome do produto for posteriormente corrigido.

## Tabela product_price_observations

| Campo ou grupo | Contrato |
| --- | --- |
| product_id, transaction_item_id | Uma observação por produto e linha de compra; o item financeiro é obrigatório e único para evitar duplicar a mesma linha. |
| place_id, occurred_on, currency | Filial/local opcional, data da compra e moeda. |
| gross_minor, discount_minor, surcharge_minor, net_minor | Snapshot reconciliável da linha; gross menos discount mais surcharge é igual a net. |
| purchased_quantity, base_quantity, base_unit_code | Quantidade original e quantidade normalizada, quando conhecida. |
| source, supersedes_id, reviewed_at | PriceObservationSource, correção versionada e revisão. |

net_minor deve coincidir com o valor líquido da linha financeira. Preço por unidade-base é cálculo exato sobre net_minor e base_quantity; arredondar só para apresentação. Sem unidade ou quantidade confiável, preservar preço total e sinalizar a observação como não comparável. Mudança de correção não apaga observação anterior.

## Tabelas price_baskets e price_basket_items

price_baskets guarda nome, moeda, período-base, revisão e arquivamento. price_basket_items guarda basket_id, product_id, posição e quantidade-base de referência positiva. Calcular a variação como custo da mesma cesta fixa no período atual dividido pelo custo no período-base. Usar somente produtos e unidades observados nos dois períodos, mostrar a cobertura da cesta e identificar resultado parcial. Sem observação suficiente, não publicar índice nem interpolar preço. Uma mudança de composição inicia nova revisão/base e mantém a série anterior identificável.

## Ligações fiscais e de estabelecimentos

A extração fiscal pode propor issuer_party_id e issuer_place_id. O emitente é a entidade empresarial; o place é o estabelecimento específico. issuer_place_id fica nullable, validado no mesmo espaço e só é preenchido automaticamente com chave/endereço suficientemente correspondentes. A coluna lógica é descrita no pacote 06; sua FK composta entra após a criação de places no pacote 13. A nota preserva sua identificação original mesmo que a relação seja corrigida.

As FKs de filiais usam places e place_identifiers do pacote 13. Fontes auxiliares de empresas não são importadas automaticamente para todas as contas; referências externas e relações aceitas guardam fonte, revisão e data.

## Integridade e comportamento

- Produto, item, observação e lugar pertencem ao mesmo space_id; todas as FKs sensíveis usam chaves compostas.
- Uma linha só gera observação após confirmação da compra e da correspondência de produto. Repetir extração ou confirmação é idempotente.
- Atribuição de desconto geral a itens é explícita ou segue regra determinística aprovada; não inferir desconto por produto sem base.
- Busca por GTIN, nome normalizado e aliases deve ser indexada por espaço; aliases não transformam variantes em um mesmo produto automaticamente.
- Aliases e identificadores são únicos somente dentro do catálogo pessoal e podem ser desativados/corrigidos com histórico; não fundir registros entre contas.
- occurred_on vem do documento/compra; data de extração e data de revisão ficam nos campos de proveniência fiscal.
- Cestas só contam produtos com unidade/moeda comparáveis. Apresentar quantidade de observações e cobertura ao lado do índice pessoal.

## Conversão do James 1.x

Preservar linhas, documentos e arquivos fiscais conforme o pacote 20. Não criar produto de catálogo automaticamente a partir de descrições incompletas ou itens sintéticos legados. A importação pode apresentar candidatos quando XML/documento trouxer código e quantidade inequívocos; a pessoa confirma o catálogo e a filial.

## Componentes previstos

MatchFiscalProducts; ReviewProductIdentity; ResolveIssuerPlace; RecordPriceObservation; CompareProductPrices; BuildPersonalPriceBasket.

## Checklist

- [ ] Revisar tipos, escala de unidades, matching e regras de correção.
- [ ] Criar migrations físicas separadas para tabelas, FKs posteriores, índices e constraints.
- [ ] Cobrir isolamento, duplicidade de observação, reconciliação e atualização histórica.
- [ ] Incluir produto/filial e procedência na auditoria, exportação e importação.
