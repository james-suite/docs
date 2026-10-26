---
id: enum-ledger-account-kind
title: Enum — LedgerAccountKind
description: Valores e invariantes propostos de LedgerAccountKind.
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
| `Asset` | `asset` | Ativo |
| `Liability` | `liability` | Passivo |
| `Expense` | `expense` | Despesa |
| `Income` | `income` | Receita |
| `Equity` | `equity` | Contrapartida patrimonial |

## Regras e transições

Proposta do livro interno condicionada ao protótipo. Define saldo natural e contrapartidas; não corresponde às categorias livres do usuário.

## Persistência

Aplicar em `ledger_accounts` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:financas) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
