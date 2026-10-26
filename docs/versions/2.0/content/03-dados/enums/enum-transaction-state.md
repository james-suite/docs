---
id: enum-transaction-state
title: Enum — TransactionState
description: Valores e invariantes propostos de TransactionState.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, financas
source_refs:
updated: 2026-10-25
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `Draft` | `draft` | Rascunho |
| `Confirmed` | `confirmed` | Confirmado |
| `Cancelled` | `cancelled` | Cancelado |

## Regras e transições

draft → confirmed ou cancelled. confirmed → cancelled exige ausência de pagamentos efetivos e de vínculos irreversíveis; se necessário, reverter tecnicamente os pagamentos antes. Compra paga que recebeu devolução comercial permanece confirmed com documento de devolução relacionado. Não esconder documento pago do relatório apenas alterando o estado. Quitação é calculada separadamente.

## Persistência

Aplicar em `financial_transactions` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:financas) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
