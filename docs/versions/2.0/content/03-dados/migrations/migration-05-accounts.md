---
id: migration-05-accounts
title: Migration 05 — Contas financeiras e contrapartidas
description: Contrato lógico de tabelas, FKs, índices e conversão: contas financeiras e contrapartidas.
type: migration-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, migrations, modelo-de-dados, enums, financas, componentes-tecnicos
source_refs:
updated: 2026-10-25
---

## Estado e ordem

Contrato proposto, sem migration PHP executável. O número é uma ordem lógica de planejamento, não timestamp de arquivo. Na implementação, gerar migrations com Artisan e separar responsabilidades; DDL e conversão dos dados não ficam no mesmo arquivo.

Dependências lógicas: 01, 03. Consulte a [ordem de migrations](doc:migrations) para resolver FKs adicionadas depois.

Campos comuns, escopo, tipos e política de exclusão seguem [Modelo de dados](doc:modelo-de-dados). As tabelas abaixo são propostas de persistência, não funcionalidades disponíveis.

## `financial_accounts`

| Campo ou grupo | Contrato |
| --- | --- |
| `name, type, currency` | Nome, FinancialAccountType e moeda fixa para saldo. |
| `institution_party_id, archived_at` | Empresa/instituição opcional e arquivamento independente da lixeira. |
| `ledger_account_id` | FK única para conta interna de ativo, condicionada ao protótipo do ledger. |

## `financial_account_identifiers`

| Campo ou grupo | Contrato |
| --- | --- |
| `account_id, scheme, label, value` | Chaves Pix e identificadores opcionais; privados, normalizados e omitidos de envios de IA. |

## `payment_methods`

| Campo ou grupo | Contrato |
| --- | --- |
| `code, label, family, archived_at` | Catálogo pessoal (Pix, dinheiro, boleto etc.); PaymentMethodFamily/handler precisa corresponder a implementação permitida. |

## `ledger_accounts`

| Campo ou grupo | Contrato |
| --- | --- |
| `kind, currency, system_code` | LedgerAccountKind e código técnico único por espaço/moeda quando aplicável. |
| `name, archived_at` | Conta interna; usuário não precisa operar um plano contábil completo. |

## Integridade, índices e comportamento

- A camada ledger é proposta técnica com gate de protótipo. Se for recusada, revisar 05/07/08 juntos antes de implementar; não manter duas fontes de saldo.
- Saldo inicial é uma operação rastreável, não coluna mutável na conta.
- Arquivamento esconde seletores de novos movimentos; manter saldos históricos, liquidações e estornos de operações antigas.
- Índices por espaço/arquivamento e restrição de mesma moeda nas alocações iniciais. Conversão entre moedas exige contrato futuro.

## Conversão do James 1.x

checking/investment/wallet preservados. Transações de saldo inicial/ajuste são convertidas por função explícita e reconciliadas, sem repetir seu efeito.

## Componentes previstos

ArchiveAccount; RestoreAccount; BalanceQuery; Money value object; guardas de instrumento arquivado.

## Checklist de implementação

- [ ] Revisar nomes, cardinalidades e invariantes com o fluxo de domínio.
- [ ] Gerar migrations, FKs, índices e constraints; verificar banco vazio e ordem real de criação.
- [ ] Criar Models, casts, factories e seeders estritamente necessários.
- [ ] Implementar a operação, Policy e validação dos IDs do mesmo espaço.
- [ ] Cobrir concorrência, transições e reconciliação aplicáveis com testes funcionais.
- [ ] Integrar auditoria, exportação/importação e estados da interface.
