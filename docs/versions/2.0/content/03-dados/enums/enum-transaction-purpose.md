---
id: enum-transaction-purpose
title: Enum — TransactionPurpose
description: Valores e invariantes propostos de TransactionPurpose.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas
source_refs:
updated: 2026-10-31
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Standard` | `standard` | Operação usual |
| `Refund` | `refund` | Devolução vinculada |
| `Interest` | `interest` | Juros |
| `Fee` | `fee` | Tarifa |

## Regras e transições

O propósito descreve a natureza do documento. Liquidação de adiantamento entre pessoas/empresas pertence a `settlement_obligations` e `settlement_payments`, não a um propósito `Reimbursement` nem a uma tag. `Refund` é devolução comercial ligada à compra original; não usar para pagamento de acerto.

## Persistência

Aplicar em `financial_transactions` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:financas) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
