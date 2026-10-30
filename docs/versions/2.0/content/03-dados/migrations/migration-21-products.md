---
id: migration-21-products
title: Migration 21 — Catálogo de produtos e preços observados
description: Contrato de persistência para produtos identificáveis, preços comparáveis e cestas pessoais.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, produtos-precos, financas, migration-03-parties, migration-06-transactions, migration-13-places, enum-price-observation-source
source_refs: https://ref.gs1.org/standards/genspecs/, https://ref.gs1.org/guidelines/2d-in-retail/
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
| package_quantity, package_unit_code, base_unit_code | Conteúdo da embalagem para apresentações fixas; unidade-base comparável (por exemplo, kg) e decimal exato, sem float. Peso comprado de item variável não fica neste cadastro. Pesos fixos diferentes identificam apresentações diferentes. |
| archived_at, notes | Arquivar sem apagar compras e observações passadas. |

## Tabelas product_identifiers e product_listings

`product_identifiers` guarda identificadores globais do produto, como GTIN. Não colocar nele RCN, PLU, SKU ou outro código interno de uma loja. A validação de GTIN inclui comprimento e dígito verificador; país de cadastro e esquema ficam explícitos quando fizerem parte da origem.

`product_listings` representa como uma empresa ou filial identifica e descreve um item em seu próprio catálogo. Campos propostos: `record_id`, `party_id` nullable, `place_id` nullable, `merchant_description`, `normalized_description`, `product_id` nullable, `valid_from`, `valid_to`, `source`, `verified_at` e estado de revisão. Exigir pelo menos um escopo (`party_id` ou `place_id`); quando ambos forem informados, validar que a filial pertence à empresa. `product_id` nullable permite manter um item local sem fingir que sua correspondência ao catálogo pessoal já foi confirmada.

`product_listing_identifiers` vincula um listing a um ou mais PLUs, SKUs ou códigos locais. Unicidade considera espaço, empresa/filial, esquema, valor normalizado e intervalo de vigência; intervalos do mesmo código no mesmo escopo não podem se sobrepor. Resolver pelo `occurred_on` da compra. O mesmo número pode ser válido em lojas diferentes; um código reutilizado com novo significado gera outro período/listing, sem mudar a associação histórica de compras. Se a data de troca não for conhecida, deixar a correspondência ambígua para revisão. GTIN fixo pertence a `product_identifiers`; em etiqueta de peso variável, o GTIN estável continua apontando ao produto e peso/preço ficam como atributos daquela linha, não como parte do GTIN.

Uma listagem de loja pode ser associada ao mesmo produto pessoal usado em outra loja, mas só após revisão suficiente de corte, marca, variante e apresentação. Nomes locais parecidos não demonstram equivalência. Não adotar catálogo global de produtos compartilhado entre contas.

## Tabela product_aliases

Guardar `product_id`, `normalized_description`, `party_id` nullable, `place_id` nullable, `valid_from`, `valid_to`, `source` e `verified_at`. Unicidade por espaço/produto/empresa/estabelecimento/descrição normalizada e vigência. `place_id` limita o alias a uma filial; `party_id` cobre a empresa quando a filial não for conhecida. Validar consistência entre Place e Party. O nome do listing comercial e o alias extraído da nota podem ser diferentes. A linha original continua preservada; aliases só sugerem correspondência, não mesclam variantes.

## Perfis e revisões de decodificação de códigos

Adicionar `place_product_code_profiles` somente para formatos que já tenham etiquetas reais e decodificação determinística validada. Campos: escopo `party_id` ou `place_id`, `profile_code` controlado, `profile_version`, `valid_from`, `valid_to`, estado, `source` e `verified_at`. O perfil seleciona uma implementação conhecida pelo James e parâmetros limitados/validados; não armazena regex ou código executável fornecido pelo usuário. Um perfil de rede só vale para todas as filiais quando essa abrangência for verificada; caso contrário, limitar ao Place.

`product_code_decodings` registra tentativa/revisão associada a `financial_transaction_item`: `raw_payload`, simbologia/sintaxe, `profile_id` nullable, versão do parser, identificador estável extraído, atributos candidatos estruturados, estado da interpretação e `reviewed_at`. Manter peso, unidade e valor aceitos nos campos financeiros tipados do item; candidatos do parser não são a fonte financeira até conferência/reconciliação. A tabela preserva uma tentativa inválida ou substituída sem sobrescrever o payload anterior. Uma leitura manual pode não ter `profile_id`.

## financial_transaction_items

Adicionar `product_id` nullable, `product_listing_id` nullable e `normalized_base_quantity`/`normalized_base_unit_code` quando a linha puder ser comparada. Ao confirmar uma correspondência de listing, salvar a identidade do produto resolvida na linha como snapshot; uma remarcação futura do listing não altera compras anteriores. As FKs compostas garantem espaço igual. A descrição e os números fiscais originais continuam imutáveis como snapshot da compra, mesmo se o nome do catálogo for posteriormente corrigido.

Para etiquetas de peso variável, guardar payload bruto e resultado da tentativa em `product_code_decodings`; separar o identificador estável local/global dos dados variáveis. A quantidade e o `quantity_unit_label` da linha registram o peso daquela compra; normalização para kg só ocorre após validar unidade, escala e precisão. Não escrever o peso da peça no cadastro permanente do produto.

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
- Busca por GTIN, nome normalizado, listings e aliases deve ser indexada por espaço; aliases não transformam variantes em um mesmo produto automaticamente.
- Primeiro identificar o tipo do código. GTIN verificado resolve o produto global; PLU/SKU/RCN procura listing vigente do Place exato e depois da Party. Para descrição textual, procurar listing/alias do Place, depois Party e só então catálogo geral. Se código e descrição indicarem entidades diferentes, apresentar conflito para revisão. Dois produtos com o mesmo rótulo em lojas diferentes continuam entidades distintas.
- Não usar o payload completo de uma etiqueta variável como chave permanente de produto: peso, preço, lote ou outros atributos podem mudar entre peças. Salvar o payload original, identidade estável, atributos decodificados, simbologia e versão do parser. Sem formato validado, guardar o payload como não interpretado e permitir revisão manual.
- Validar dígito verificador quando aplicável, sintaxe e identificadores de aplicação GS1, separadores, escala decimal, unidade, sinal, limites plausíveis e coerência entre dados codificados e legíveis na etiqueta. Não assumir que RCN é GTIN nem generalizar um perfil local. A política de arredondamento precisa ser explícita; divergência fora da tolerância não é corrigida automaticamente.
- Quantidade × preço unitário deve reconciliar com o total líquido da linha dentro da regra monetária. Se a etiqueta traz preço total e peso, tratar o total como valor da linha e não multiplicá-lo novamente; se peso ou unidade estiverem ausentes, não produzir comparação por kg. Desconto de cabeçalho ou fidelidade só compõe o preço unitário quando atribuível à linha; registrar preço de etiqueta e preço efetivamente pago separadamente se ambos forem necessários.
- Nota fiscal e etiqueta podem descrever a mesma compra. Associá-las à mesma linha financeira e garantir uma observação de preço por linha; repetição de captura/importação deve ser idempotente.
- Uma alteração posterior de produto, listing, alias ou perfil não reescreve identidade, quantidade e valores já confirmados. Correção da linha cria revisão/auditoria e invalida apenas projeções recalculáveis, como comparações futuras.
- Aliases e identificadores são únicos somente dentro do catálogo pessoal e podem ser desativados/corrigidos com histórico; não fundir registros entre contas.
- occurred_on vem do documento/compra; data de extração e data de revisão ficam nos campos de proveniência fiscal.
- Cestas só contam produtos com unidade/moeda comparáveis. Apresentar quantidade de observações e cobertura ao lado do índice pessoal.

## Conversão do James 1.x

Preservar linhas, documentos e arquivos fiscais conforme o pacote 20. Não criar produto de catálogo automaticamente a partir de descrições incompletas ou itens sintéticos legados. A importação pode apresentar candidatos quando XML/documento trouxer código e quantidade inequívocos; a pessoa confirma o catálogo e a filial.

## Componentes previstos

MatchFiscalProducts; ReviewProductIdentity; ResolveIssuerPlace; MatchProductListing; DecodeVariableMeasureCode; ReviewProductCodeDecoding; RecordPriceObservation; CompareProductPrices; BuildPersonalPriceBasket. A decodificação automática é fase posterior à coleta de amostras e validação dos perfis suportados.

## Checklist

- [ ] Revisar tipos, escala de unidades, escopo temporal de códigos locais, matching e regras de correção.
- [ ] Reunir etiquetas reais de mais de uma loja, com peso/preço conhecidos, antes de aprovar qualquer perfil automático.
- [ ] Criar migrations físicas separadas para tabelas, FKs posteriores, índices e constraints.
- [ ] Cobrir isolamento, códigos repetidos em Places diferentes, reutilização temporal de PLU, leitura ambígua, duplicidade de observação, reconciliação e atualização histórica.
- [ ] Incluir produto/filial e procedência na auditoria, exportação e importação.
