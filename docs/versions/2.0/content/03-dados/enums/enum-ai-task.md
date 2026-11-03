---
id: enum-ai-task
title: Enum — AiTask
description: Valores e invariantes propostos de AiTask.
type: enum-reference
status: proposed
visibility: public
tags: james/2.0, planejamento, dados
related: planejamento-2-0, enums, migrations, ia-e-memoria, assistente-pessoal-rag, agenda-tarefas, financas
source_refs:
updated: 2026-11-01
---

## Contrato proposto

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `TagClassification` | `tag_classification` | Classificar separadamente uma transação ou item com tags específicas; quando não houver tag adequada, propor uma nova para criação e atribuição junto com a aplicação autorizada do resultado. |
| `FiscalExtraction` | `fiscal_extraction` | Extrair candidatos estruturados de documento fiscal quando um leitor determinístico não atender e o usuário permitir o envio |
| `CalendarCommand` | `calendar_command` | Interpretar um pedido explícito de agenda e preparar uma operação tipada de compromisso, sujeita à autorização da função e às confirmações do domínio |
| `TransactionDraft` | `transaction_draft` | Interpretar uma descrição natural sem documento fiscal e criar um rascunho financeiro para revisão, sem efetivar pagamentos |

### Cases candidatos para fase futura do assistente

Não fazem parte da enum inicial nem devem ser persistidos antes da fase de [Assistente pessoal e RAG](doc:assistente-pessoal-rag):

| Case candidato | Valor persistido | Significado |
| --- | --- | --- |
| `AssistantAnswer` | `assistant_answer` | Responder pergunta com evidências de fontes autorizadas e referências ao registro. |
| `MemoryIndexing` | `memory_indexing` | Gerar/atualizar trechos e embeddings derivados para fontes que a pessoa incluiu. |
| `AssistantActionProposal` | `assistant_action_proposal` | Preparar proposta tipada por ferramenta permitida, sem efetivá-la. |

## Regras e transições

TagClassification e FiscalExtraction são as funções definidas. CalendarCommand e TransactionDraft são propostas adicionais em avaliação e exigem permissões separadas. TransactionDraft só cria estado Draft em `financial_transactions`; não escolhe nem efetiva pagamento, não cria documento fiscal e não altera categorias. A enum inicial não habilita chat geral, extração de fatos pessoais nem ações autônomas fora dos comandos tipados. As cases candidatas de assistente só entram após aprovar os respectivos contratos, controles e avaliações. FiscalExtraction só processa a nota vinculada à execução, sob permissão e orçamento próprios; não confirma compra, pagador ou pagamento.
