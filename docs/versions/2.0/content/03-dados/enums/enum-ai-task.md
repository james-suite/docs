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
updated: 2026-10-29
---

## Contrato proposto

Esta nota planeja uma enum string; a classe PHP ainda não existe no 2.0. Cases usam TitleCase e valores persistidos estáveis. A implementação e as versões dos pacotes serão escolhidas em 2027.

| Case | Valor persistido | Significado |
| --- | --- | --- |
| `TagSuggestion` | `tag_suggestion` | Sugerir tags para uma transação ou um item de transação; pode propor nome novo, sempre sujeito à confirmação antes de criar a tag. |
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

TagSuggestion e FiscalExtraction são as funções definidas. CalendarCommand e TransactionDraft são propostas adicionais em avaliação e exigem permissões separadas. TransactionDraft só cria estado Draft em `financial_transactions`; não escolhe nem efetiva pagamento, não cria documento fiscal e não altera categorias. A enum inicial não habilita chat geral, extração de fatos pessoais nem ações autônomas fora dos comandos tipados. As cases candidatas de assistente só entram após aprovar os respectivos contratos, controles e avaliações. FiscalExtraction só processa a nota vinculada à execução, sob permissão e orçamento próprios; não confirma compra, pagador ou pagamento.

## Persistência

Aplicar em `ai_runs` conforme [migrations](doc:migrations). Persistir string com cast Eloquent e restrição de valores quando apropriado. Não usar texto traduzido como valor de banco. Valores derivados ou exibidos como filtro não devem ser persistidos novamente sem necessidade.

## Checklist

- [ ] Revisar os cases no fluxo de [domínio](doc:ia-e-memoria) e remover capacidades ainda não implementadas da UI.
- [ ] Criar enum, rótulos e cast no Model correspondente.
- [ ] Proteger transições na operação de domínio e no banco quando houver invariante local.
- [ ] Validar autorização, valores desconhecidos, concorrência e conversão de dados legados quando aplicável.
