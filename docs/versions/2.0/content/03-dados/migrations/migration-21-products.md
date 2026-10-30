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
updated: 2026-10-29
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
| package_quantity, package_unit_code, base_unit_code | Conteúdo da embalagem opcional para produtos de peso variável; unidade-base comparável (por exemplo, kg) e decimal exato, sem float. Pesos fixos diferentes identificam apresentações diferentes. |
| archived_at, notes | Arquivar sem apagar compras e observações passadas. |

## Tabela product_identifiers

| Campo ou grupo | Contrato |
| --- | --- |
| product_id, scheme, country, value_normalized, party_id, place_id | Identificador como GTIN/EAN/UPC global ou código local limitado a empresa/estabelecimento. party_id e place_id definem escopo opcional; validar que o estabelecimento pertence à empresa quando ambos forem informados. |
| source, verified_at | Origem e estado de conferência do código. |

Unicidade proposta por espaço, esquema, país, código normalizado e escopo. GTIN/EAN conhecido é global no catálogo pessoal; PLU, SKU e códigos de balança são locais e podem se repetir em outros Places. Conflito no mesmo escopo ou embalagem diferente precisa de revisão. Não adotar catálogo global de produtos no primeiro corte.

## Tabela product_aliases

Guardar product_id, normalized_description, party_id nullable, place_id nullable, source e verified_at. Unicidade por espaço/produto/empresa/estabelecimento/descrição normalizada. place_id limita o alias a uma filial específica; party_id cobre a empresa quando a filial não for conhecida. Validar consistência do Place com Party. A linha original continua preservada. Aliases só sugerem correspondência, não mesclam variantes.

## financial_transaction_items

Adicionar product_id nullable e normalized_base_quantity/normalized_base_unit_code quando a linha puder ser comparada. A FK composta garante espaço igual. A descrição e os números fiscais originais continuam imutáveis como snapshot da compra, mesmo se o nome do produto for posteriormente corrigido.

Para etiquetas de peso variável, preservar opcionalmente `raw_scanned_identifier` e separar o código PLU do peso/preço decodificado. A quantidade e o `quantity_unit_label` da linha registram o peso daquela compra; normalização para kg só ocorre após validar a unidade. Não escrever o peso da peça no cadastro permanente do produto.

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
- Uma linha só gera observação após confirmação da compra e da correspondência de produto. ManualEntry pode vir de linha sem nota criada por pessoa ou preparada por IA, após confirmação de produto, valor, quantidade/unidade e lugar conhecido. Repetir extração ou confirmação é idempotente.
- Atribuição de desconto geral a itens é explícita ou segue regra determinística aprovada; não inferir desconto por produto sem base.
- Busca por GTIN, nome normalizado e aliases deve ser indexada por espaço; aliases não transformam variantes em um mesmo produto automaticamente.
- Resolver códigos globais verificados e, para códigos/descrições locais, procurar primeiro o Place exato, depois Party e por último correspondência geral. Dois produtos com o mesmo rótulo em lojas diferentes continuam entidades distintas; resultado ambíguo exige escolha.
- Se for necessário decodificar etiquetas que embutem PLU, peso ou preço, usar perfil de leitura versionado associado ao estabelecimento, com identificador de perfil controlado pelo código e amostras verificadas. Não aceitar regex/código executável fornecido pelo usuário nem aplicar formato de uma loja a outra. Perfil ainda não validado deixa a etiqueta para revisão manual.
- Quantidade × preço unitário deve reconciliar com o total da linha dentro da regra monetária. Se a etiqueta traz preço total e peso, tratar o total como fonte da linha e não multiplicá-lo novamente; se peso ou unidade estiverem ausentes, não produzir comparação por kg.
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
