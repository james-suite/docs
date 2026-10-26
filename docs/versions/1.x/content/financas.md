---
id: financas
title: Finanças
description: Estrutura relacional para contas, transações, cartões, recorrências, tags e relatórios.
type: feature
status: observed
visibility: public
tags: finanças, domínio, relatórios
related: funcionalidades, contas-financeiras, transacoes, cartoes, recorrencias, relatorios
source_refs: https://github.com/james-suite/james/blob/master/routes/financial.php, https://github.com/james-suite/james/tree/master/app/Models
---

### Visão Geral
O módulo Financeiro é o núcleo de controle patrimonial do James.
A regra de ouro é estruturação relacional — dados financeiros precisam
ser somados, agrupados e cruzados em relatórios. O uso de JSONB foi
descartado para garantir integridade e performance nas consultas.

### Contas (`financial_accounts`)
Tabela única para todas as carteiras — dinheiro físico (`wallet`), conta corrente (`checking`)
e investimentos (`investment`). Sem tabelas separadas de bancos.
Ícones dinâmicos via `FinancialAccountType` e suporte a chaves Pix.

### Transações — Estrutura Pai e Filho
- `financial_transactions` (Pai) — o registro do pagamento na totalidade:
  conta, valor, data, cartão, status (`TransactionStatus`: Draft, Pending, Posted) e anexos.
- `financial_transaction_items` (Filhos) — detalhamento dos itens,
  especialmente útil para itens de nota fiscal. Garante relatórios
  precisos por item.

### Cartão de Crédito
Controle de faturas (`financial_credit_card_invoices`) com data de fechamento e vencimento (ajustadas por feriados via BrasilAPI).
Status gerenciado via `InvoiceStatus` (Paid, PartiallyPaid, Open, Overdue, Closed).
Gastos no cartão não afetam o saldo imediatamente — a saída ocorre quando a fatura é paga.

### Parcelamentos e Recorrências
Compras parceladas geram parcelas futuras automaticamente vinculadas à
transação original. Transações recorrentes (salário, aluguel, assinaturas)
são processadas via Laravel Scheduler diariamente.

### Transferências entre Contas
Gera duas transações vinculadas (`transfer_pair_id`) — saída na conta A e entrada na conta B.
O saldo total não é afetado.

### Sistema de Tags
Substitui o sistema rígido de categorias. Tabela `financial_tags` com `name`, `icon` (Heroicons, Tabler e Phosphor) e `color_hex`.
Tabela pivot polimórfica `financial_taggables` com `is_primary` — permite vincular tags tanto
na transação pai quanto em itens filhos específicos.

### Relatórios e Dashboards (Apache ECharts)
Visualizações analíticas em Regime de Caixa (Evolução de Saldo, Diagrama de Sankey do fluxo de dinheiro, drill-down dinâmico por tags e isolamento de investimentos).

### Importação de NFC-e
Importação assíncrona de notas fiscais de consumidor a partir de URLs públicas ou da leitura do QR Code pela câmera, inicialmente pelo portal SVRS. A nota é criada como rascunho, com emitente, documento, itens, descontos e metadados fiscais, sem afetar os cálculos financeiros até a revisão. Falhas podem ser reenviadas pela notificação recebida pelo usuário.

Consulte a [documentação da Importação de NFC-e](doc:nfce).

### Integrações
- **Módulo Acertos** — pagamentos e liquidações refletem como transações financeiras.
- **Módulo Notificações** — avisos de faturas fechadas, lembretes de vencimento e alertas via Telegram / E-mail.
- **Módulo de Auditoria** — rastreabilidade de todas as alterações em transações, contas e cartões.

### Referências
- [Roadmap — Módulo Finanças](doc:roadmap)
- [Decisão 007 — Padronização de data e moeda](doc:decisoes)
- [Decisão 014 — Biblioteca gráfica (Apache ECharts)](doc:decisoes)
