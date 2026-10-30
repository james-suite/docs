---
id: produtos-precos
title: Produtos, histórico de preços e minha inflação
description: Catálogo pessoal de produtos, preços observados em compras e comparação ao longo do tempo e entre lugares.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, financas, ia-e-memoria, pessoas-empresas, lugares-checkins, veiculos, migration-21-products, migration-06-transactions
source_refs: https://ref.gs1.org/standards/genspecs/, https://ref.gs1.org/guidelines/2d-in-retail/
updated: 2026-10-29
---

## Compras como fonte do histórico

Uma linha de produto extraída de nota fiscal pode alimentar o catálogo pessoal e o histórico de preços depois de conferida. Cada observação aponta para a linha financeira original, para o produto identificado e, quando resolvido, para a filial ou lugar da compra. Assim, o preço não vira uma segunda despesa nem se perde a ligação com o comprovante.

O catálogo pertence ao espaço pessoal. Nome, marca, variante, apresentação e códigos como GTIN/EAN ajudam a reconhecer produtos; esquema e país do identificador ficam explícitos. O mesmo nome pode representar tamanhos ou versões diferentes. Se a linha não trouxer código, o James sugere uma correspondência por descrição, marca, embalagem e estabelecimento e pede revisão quando houver ambiguidade. Não unir produtos diferentes apenas porque os nomes se parecem.

## Preços comparáveis

Guardar a quantidade comprada e sua unidade de medida, além do valor bruto, desconto atribuível à linha e valor líquido. Converter embalagens para uma unidade base conhecida — por exemplo, preço por kg, litro ou unidade — com decimal exato. Quando a unidade ou o conteúdo da embalagem forem desconhecidos, mostrar o preço pago mas não afirmar uma comparação normalizada.

O histórico registra a data fiscal da compra, o preço observado, o produto, a filial/local quando conhecido e a origem da informação. Uma alteração posterior no nome do catálogo não modifica o texto e os valores originais da nota. Correções criam uma revisão rastreável da observação.

Desconto de cabeçalho só entra no preço do produto quando estiver atribuído explicitamente a uma linha ou quando uma regra segura já o reconciliar. Não distribuir automaticamente desconto geral entre vários produtos. Promoções, quantidade, peso e preço por embalagem precisam continuar distinguíveis.

## Filial, empresa e lugar

Uma empresa (Party) pode operar muitos estabelecimentos (Place). Cada lugar guarda endereço e, se disponível, identificadores próprios da filial. A nota pode apontar para a empresa e para a filial separadamente. CNPJ de filial e endereço fiscal ajudam a resolver o posto ou mercado exato; coordenadas geográficas são outro dado e podem exigir geocodificação ou confirmação.

O James pode sugerir um lugar a partir do emitente, endereço e identificadores extraídos, procurando primeiro os registros já confirmados pelo usuário. Um nome de empresa sozinho não basta para escolher entre filiais. Criar lugar ou associação com baixa confiança fica como sugestão revisável. Essa compra não cria check-in confirmado.

Um produto pode ter código ou nome de venda que só faz sentido em uma loja/filial. O catálogo continua sendo pessoal, mas aliases e identificadores podem ser limitados à empresa (`Party`) ou ao estabelecimento (`Place`). Reconhecer primeiro códigos globais verificados; para códigos locais e descrições, priorizar correspondência exata no estabelecimento, depois na empresa e só então correspondência geral. Se a loja não for conhecida ou houver mais de um produto compatível, mostrar opções em vez de escolher por semelhança. Esse escopo vale tanto para itens fiscais quanto para linhas digitadas ou preparadas por IA.

Uma linha digitada sem nota ou criada inicialmente por IA pode formar observação `ManualEntry` depois que a pessoa confirmar produto, valor, quantidade/unidade e local conhecido. O rótulo original e a origem manual permanecem registrados; o fato de a IA ter ajudado a preparar a transação fica na proveniência/auditoria.

## Carnes e etiquetas de peso variável

Separar três identidades que podem aparecer na mesma compra:

1. **Produto comparável do catálogo pessoal:** por exemplo, “patinho bovino moído”. Marca, corte, preparo, teor de gordura e apresentação podem mudar o que é comparável; não unir itens apenas porque o nome é parecido.
2. **Cadastro comercial da loja:** nome de venda, SKU ou PLU usado por uma empresa/filial. Ele pode ser ligado a um produto do catálogo após confirmação, ou ficar sem correspondência até haver informação suficiente. A mesma peça pode ter nomes/códigos diferentes em duas lojas; o mesmo PLU também pode significar coisas diferentes em lojas distintas.
3. **Unidade física comprada:** a peça/pacote específico daquela transação, com seu peso e preço. Peso variável nunca vira peso fixo do cadastro do produto.

Um GTIN de embalagem fixa identifica uma apresentação vendida — por exemplo, 500 g e 1 kg são variantes distintas. Já etiquetas de produtos pesados podem usar um código restrito/local (RCN/PLU) ou um GTIN com atributos adicionais. No padrão GS1, RCN não é GTIN e não deve ser tratado como código global; códigos RCN e seus formatos dependem do ambiente/território. Alguns códigos estruturados GS1 podem carregar o GTIN e atributos como peso líquido, preço, lote ou validade. O James deve distinguir a identidade estável do produto dos dados variáveis da etiqueta, sem assumir que toda sequência numérica segue um formato universal. ([GS1 General Specifications](https://ref.gs1.org/standards/genspecs/), [guia GS1 para códigos 2D no varejo](https://ref.gs1.org/guidelines/2d-in-retail/))

No produto fixo, peso e unidade da embalagem ficam no catálogo/variante. Na linha variável, registrar quantidade pesada com unidade e precisão explícitas, preço unitário quando conhecido e total líquido da linha. Manter separados preço por kg e total pago: se o código já informar o total, não multiplicá-lo novamente; se informar peso e preço unitário, reconciliar o produto desses valores com a linha fiscal, respeitando o arredondamento monetário. Se só houver o valor total, não inventar peso nem preço por kg.

Ao ler etiqueta, preservar o conteúdo bruto e a simbologia quando disponível, o código estável extraído, os atributos decodificados e a versão do perfil determinístico usado. Nunca usar o código completo que muda a cada peça como identificador permanente do produto. Validar estrutura, dígito verificador quando aplicável, unidade, escala decimal, campos obrigatórios e coerência com o total. Perfil ausente, código malformado ou divergência vira proposta para revisão manual, sem preço comparável automático.

Perfis de leitura precisam ser associados à empresa ou filial correta, ter versão e exemplos reais conferidos. Regras desconhecidas não devem ser inferidas por prefixos genéricos; não aceitar expressões/código arbitrário como parser. Primeiro registrar PLU/SKU local e fazer revisão manual; adicionar decodificação automática apenas para formatos que tenham exemplos suficientes e validação determinística. A IA pode ajudar a ler texto da nota no fluxo fiscal autorizado, mas não é decodificador de código de barras nem pode inventar peso, corte ou preço.

Exemplo: “PLU 1234 — patinho moído” de um mercado e “PLU 076 — carne bovina moída” de outro são dois cadastros locais. A pessoa pode relacioná-los ao mesmo produto comparável se confirmar corte e preparo; cada compra continua guardando seu próprio nome, código, peso e valor.

Se a mesma compra aparecer na nota e na etiqueta fotografada/lida, associar as duas fontes à mesma linha e criar uma única observação de preço. Uma correção futura do alias, da loja ou do perfil não reescreve a compra passada; a linha mantém o produto, o cadastro comercial e a interpretação confirmados naquela revisão. Descrição, loja, código, unidade, preço/kg e total precisam estar visíveis durante a conferência. Sem perfil ou peso confiável, guardar a descrição/código originais e pedir revisão.

Nesta etapa, o objetivo é identidade de produto e histórico de compra/preço. Campos extras encontrados na etiqueta não ativam automaticamente rastreabilidade de lote, validade ou controle de estoque.

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
