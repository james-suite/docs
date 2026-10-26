---
id: produtos-precos
title: Produtos, histórico de preços e minha inflação
description: Catálogo pessoal de produtos, preços observados em compras e comparação ao longo do tempo e entre lugares.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, financas, pessoas-empresas, lugares-checkins, veiculos, estoque-despensa, migration-21-products, migration-22-stock, migration-06-transactions
source_refs:
updated: 2026-10-26
---

## Compras como fonte do histórico

Uma linha de produto extraída de nota fiscal pode alimentar o catálogo pessoal e o histórico de preços depois de conferida. Cada observação aponta para a linha financeira original, para o produto identificado e, quando resolvido, para a filial ou lugar da compra. Assim, o preço não vira uma segunda despesa nem se perde a ligação com o comprovante.

O catálogo pertence ao espaço pessoal. Nome, marca, variante, apresentação e códigos como GTIN/EAN ajudam a reconhecer produtos; esquema e país do identificador ficam explícitos. O mesmo nome pode representar tamanhos ou versões diferentes. Se a nota não trouxer código, o James sugere uma correspondência por descrição, marca e embalagem e pede revisão quando houver ambiguidade. Não unir produtos diferentes apenas porque os nomes se parecem.

## Preços comparáveis

Guardar a quantidade comprada e sua unidade de medida, além do valor bruto, desconto atribuível à linha e valor líquido. Converter embalagens para uma unidade base conhecida — por exemplo, preço por kg, litro ou unidade — com decimal exato. Quando a unidade ou o conteúdo da embalagem forem desconhecidos, mostrar o preço pago mas não afirmar uma comparação normalizada.

O histórico registra a data fiscal da compra, o preço observado, o produto, a filial/local quando conhecido e a origem da informação. Uma alteração posterior no nome do catálogo não modifica o texto e os valores originais da nota. Correções criam uma revisão rastreável da observação.

Desconto de cabeçalho só entra no preço do produto quando estiver atribuído explicitamente a uma linha ou quando uma regra segura já o reconciliar. Não distribuir automaticamente desconto geral entre vários produtos. Promoções, quantidade, peso e preço por embalagem precisam continuar distinguíveis.

## Vínculo opcional com estoque

Uma compra confirmada pode sugerir a entrada de uma linha no [estoque e despensa](doc:estoque-despensa), mas a pessoa decide quais itens controlar e em qual quantidade/local/lote. O produto do catálogo pode ser associado ao StockItem; isso não acontece só por a linha ter sido comprada. Entrada de estoque não duplica preço nem despesa, e itens no estoque também podem existir sem linha financeira.

## Filial, empresa e lugar

Uma empresa (Party) pode operar muitos estabelecimentos (Place). Cada lugar guarda endereço e, se disponível, identificadores próprios da filial. A nota pode apontar para a empresa e para a filial separadamente. CNPJ de filial e endereço fiscal ajudam a resolver o posto ou mercado exato; coordenadas geográficas são outro dado e podem exigir geocodificação ou confirmação.

O James pode sugerir um lugar a partir do emitente, endereço e identificadores extraídos, procurando primeiro os registros já confirmados pelo usuário. Um nome de empresa sozinho não basta para escolher entre filiais. Criar lugar ou associação com baixa confiança fica como sugestão revisável. Essa compra não cria check-in confirmado.

## “Minha inflação”

Com observações comparáveis, oferecer evolução de preço por produto, comparação entre lugares e uma cesta pessoal chamada “Minha inflação”. A pessoa escolhe produtos e quantidades/pesos de referência; o James compara períodos usando somente itens identificados e unidades compatíveis. Exibir cobertura, período-base, substituições e ausências ao lado do índice.

Esse indicador descreve as compras observadas daquela pessoa e daquela cesta. Não é o IPCA, não representa toda a renda ou consumo e não deve preencher meses sem compras com preços inventados. Alterar a cesta cria uma nova revisão/base de comparação, em vez de reescrever a série anterior.

## Relações empresariais e bases auxiliares

O cadastro local relaciona empresa, filial, marca, grupo e prestador com tipo de relação, vigência, fonte e grau de confirmação. Uma base auxiliar de empresas pode futuramente sugerir razão social, identificadores, filiais ou vínculos empresariais. A origem, data de consulta e versão dos dados precisam acompanhar cada sugestão; aceitar uma relação externa não altera o histórico fiscal.

Separar claramente o catálogo privado do usuário dos dados auxiliares compartilhados da instalação. Não usar notas fiscais de uma pessoa para enriquecer a base visível a outras contas. Antes de escolher provedores, confirmar fonte, licença, atualização, cobertura e custo; sem isso, manter busca e relações locais. Dados externos não fundem empresas automaticamente.

## Etapas propostas

1. Capturar itens, identificadores e unidades de notas; corrigir correspondência e filial durante a revisão fiscal.
2. Consultar histórico de um produto e comparar unidades/preços em lugares diferentes.
3. Com volume e qualidade suficientes, permitir cestas pessoais e o índice “Minha inflação”.
4. Avaliar fontes auxiliares de empresas e vínculos, com proveniência e revisão explícitas.
