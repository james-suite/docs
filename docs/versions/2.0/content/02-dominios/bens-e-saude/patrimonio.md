---
id: patrimonio
title: Patrimônio e bens
description: Bens, avaliações e documentos sem duplicar valores financeiros ou de veículos.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, veiculos, financas, migrations
source_refs:
updated: 2026-10-25
---

## Contrato

Guardar bem, categoria configurável, estado de posse, datas de aquisição/alienação, preço de aquisição e avaliações datadas com fonte. Associar empresa/pessoa da compra, anexos e transação existente. O veículo pode existir e ter histórico completo sem ser cadastrado como patrimônio; se o usuário quiser acompanhar seu valor, pode vinculá-lo a um único bem, sem duplicar cadastro nem custo.

Avaliação patrimonial é uma observação; valorização ou depreciação não representa entrada/saída de dinheiro. O total patrimonial precisa declarar se usa custo ou avaliação e sua data. Contas de investimento já incluídas no financeiro não devem aparecer de novo como bem equivalente.

Manutenção, garantia, número de série e localização atual são extensões úteis, opcionais. Histórico de propriedade permanece após arquivar o bem. Alienação pode vincular uma receita real, sem fabricar uma automaticamente.
