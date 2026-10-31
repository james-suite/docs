---
id: assistente-pessoal-rag
title: Assistente pessoal, memória e RAG
description: Evolução futura da IA do James para buscar respostas fundamentadas e executar ações autorizadas.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio, IA
related: ia-e-memoria, diario, lugares-checkins, financas, veiculos, modelo-de-dados, migration-12-ai, componentes-tecnicos, fases-2-0, decisoes-e-pendencias
source_refs: https://platform.openai.com/docs/guides/embeddings, https://github.com/pgvector/pgvector
updated: 2026-10-29
---

## Visão

O assistente pessoal do James é uma evolução das funções de IA delimitadas: entende perguntas sobre os dados escolhidos pela pessoa e ajuda a agir nos módulos. Isso não é requisito do primeiro lançamento migrável do 2.0. O núcleo do James precisa continuar funcionando sem IA, e os registros tipados continuam sendo a fonte oficial dos fatos.

O assistente não ganha acesso irrestrito ao banco. Ele recebe ferramentas e fontes autorizadas para a tarefa atual; o servidor aplica Policies, espaço pessoal, permissões, limites e auditoria. Uma resposta que não encontra evidência suficiente diz isso claramente, em vez de completar lacunas com palpite.

## Capacidades em camadas

| Camada | Comportamento | Gate para avançar |
| --- | --- | --- |
| 1. Funções delimitadas | Sugestões de tags, fallback fiscal e, se aprovados, rascunho de transação e comando de agenda. | Revisão, orçamento e autorização independentes por função. |
| 2. Busca fundamentada (RAG) | Perguntas em linguagem natural sobre fontes que a pessoa selecionou; resposta com links para registros, período e ressalvas. Somente leitura. | Isolamento validado, fontes atualizáveis/apagáveis, busca avaliada com perguntas representativas. |
| 3. Ferramentas de domínio | Consultar valores calculados pelo domínio e preparar rascunhos/propostas de ações por módulos. | Ferramenta allowlisted, política do módulo e confirmação proporcional ao efeito. |
| 4. Assistência proativa | Resumos, lembretes ou sugestões iniciados por preferências configuradas pela pessoa. | Opt-in separado, frequência/quiet hours, explicação da origem, controle de custo e desligamento fácil. |

Voz, automação em segundo plano e ações autônomas ficam como decisões futuras separadas. A visão de assistente não autoriza monitoramento contínuo, captura de localização extra, execução financeira ou envio de mensagens sem um fluxo próprio aprovado.

## Como o RAG funcionaria

RAG (*Retrieval-Augmented Generation*) combina recuperação de fontes e geração de resposta. Para uma pergunta como “o que aconteceu com o carro antes de eu trocar o óleo?”, o James localiza diário, serviços e compras relevantes, envia trechos autorizados ao modelo e apresenta uma resposta ligada às respectivas telas.

1. O roteador identifica se a pergunta é uma busca textual, uma consulta numérica, uma ação ou algo não suportado.
2. O servidor resolve espaço, módulos incluídos e permissões antes de recuperar qualquer contexto.
3. Busca híbrida combina os meios adequados: SQL/Actions para valores e relações tipadas, busca textual para termos exatos e busca vetorial para significado em texto livre.
4. Remove duplicatas, limita fontes, período e tamanho do contexto; preserva data, fuso, estado e IDs de origem.
5. O modelo recebe somente as evidências escolhidas e devolve resposta estruturada com referências de origem.
6. O James valida as referências e abre cada registro; sem evidência suficiente, responde que não encontrou dados.

RAG não substitui consultas financeiras. “Quanto gastei com combustível em setembro?” usa a consulta de domínio com moeda, estado e período corretos. “Que posto eu comentei que era bom?” pode usar busca textual/vetorial no diário. Uma pergunta que mistura os dois pode combinar ferramentas e apresentar separadamente cálculo e contexto narrativo.

## Memória e busca vetorial

Manter três coisas distinguíveis:

- **Fonte factual:** transações, visitas, eventos, veículos, notas e revisões do diário persistidos nos módulos existentes.
- **Índice derivado:** trechos, resumos de busca e embeddings reconstruíveis que apontam para fonte e revisão. Embedding é uma lista numérica usada para comparar proximidade semântica; não é um fato novo nem substitui o texto original.
- **Memória confirmada:** preferências ou fatos pessoais que a pessoa vê, corrige, confirma e apaga. Inferências do modelo ficam como candidatos até essa confirmação; fatos temporais guardam validade e fonte.

Começar avaliando busca textual do PostgreSQL e consultas estruturadas. Se perguntas naturais sobre notas do diário continuarem difíceis, testar a extensão `pgvector` no mesmo PostgreSQL antes de operar outro banco. Busca exata serve como baseline e para volumes modestos; um índice aproximado como HNSW só entra após medir latência e cobertura. A dimensão do vetor precisa corresponder ao modelo escolhido; registrar o identificador/versão do modelo e da divisão em trechos para permitir reindexação.

Contrato lógico futuro, não migration obrigatória do núcleo:

- `ai_memory_sources`: `space_id`, `record_id`, revisão da fonte, hash do conteúdo indexável, módulo, estado de inclusão/indexação, versão do conversor, modelo de embeddings e `indexed_at`. Chave única pela fonte/revisão/perfil de indexação.
- `ai_memory_chunks`: `space_id`, `memory_source_id`, número do trecho, snapshot textual mínimo, embedding `vector(N)`, modelo/dimensão, versão do chunker e contagem aproximada de tokens. Índices relacionais por espaço/fonte/revisão; índice vetorial separado, se necessário.
- `ai_conversations` e `ai_messages`: conversas visíveis à pessoa e mensagens finais necessárias para retomar a sessão. Retenção própria; prompts internos, segredos e payloads completos de ferramentas não são guardados por padrão.
- `assistant_action_proposals`: quando ações forem autorizadas, ação/tipo allowlisted, alvo e revisão esperada, parâmetros tipados, expiração, autor da confirmação, estado e chave de idempotência. Não aceitar SQL, nome de tabela ou classe vindo do modelo.

As propostas especializadas já planejadas para agenda e transações continuam usando seus contratos de domínio. Uma futura camada genérica de ferramenta pode orquestrá-las, mas não criar um segundo registro de proposta financeira ou de evento para a mesma operação.

Não indexar tudo por padrão. A pessoa escolhe módulos/fontes permitidos; diário íntimo, saúde, finanças e coordenadas precisam de consentimentos/controles próprios. O índice e os resumos são dados privados derivados: revogar acesso, editar ou apagar uma fonte invalida seus trechos, vetores, caches e resumos. Jobs de indexação verificam a revisão atual e são idempotentes, evitando deixar conteúdo antigo após uma edição concorrente.

## Isolamento e recuperação

Toda busca deriva `space_id` da sessão autenticada no servidor. Nunca confiar em IDs, permissões ou escopo sugeridos pela mensagem ou resposta do modelo. Filtrar fontes autorizadas antes de montar contexto e validar novamente cada referência/Action.

Em busca vetorial aproximada, os filtros de espaço/módulo também afetam quantos candidatos são encontrados; recall e isolamento precisam ser medidos. O filtro SQL/RLS é uma barreira de autorização, não uma otimização opcional. Comparar resultados aproximados a busca exata em amostras, ajustar índice ou particionar somente se medições justificarem.

Textos recuperados são conteúdo não confiável: uma nota pode conter instruções maliciosas ou acidentais, que devem ser tratadas como dados citados, nunca como instruções para o assistente. Nenhuma informação de outro usuário pode aparecer em respostas, logs, caches ou resultados de busca.

## Ferramentas e ações

Ferramentas de leitura chamam consultas de domínio autorizadas; não fornecem conexão SQL direta ao modelo. Exemplos: `SearchJournalEntries`, `GetVehicleHistory`, `GetSpendingSummary` e `FindCalendarEvents`. A ferramenta financeira devolve cálculo determinístico e período aplicado, não pede ao LLM para somar valores manualmente.

Ferramentas de escrita passam pelo módulo dono do dado e criam propostas tipadas. Rascunhos podem ser editados antes da confirmação; pagamentos, transferências, exclusões, mensagens externas e alterações irreversíveis nunca são efetivados só porque o modelo os pediu. A confirmação revalida alvo, revisão, autorização e idempotência; a auditoria registra pedido, ferramenta, resultado e quem confirmou, sem salvar cadeia de pensamento.

## Desempenho, custo e qualidade

- Usar regras e consultas locais antes de chamar um modelo; escolher o menor fluxo/modelo que atende à tarefa, sem fallback invisível.
- Indexar em fila após mudanças, apenas a fonte/revisão afetada; agrupar chamadas quando o provedor suportar, e reservar orçamento também para embeddings/reindexações.
- Cachear somente respostas recuperáveis e ligadas a `space_id`, revisão de fontes, política e versão do modelo; invalidar na mudança de qualquer dependência.
- Limitar fontes/trechos/contexto, tempo de execução e gasto por função; permitir desativar IA ou uma fonte sem perder o uso manual do James.
- Avaliar conjunto de perguntas autorizadas/sintéticas com fontes esperadas: recall das fontes, correção da resposta, suporte das citações, abstenção quando não há evidência, latência, custo e tentativa de acesso cruzado.
- Comparar busca vetorial/híbrida contra busca textual/SQL e manter a opção mais simples se a melhora não justificar custo, operação e exposição de dados.

Guardar telemetria mínima: `ai_run_id`, tarefa, conexão/modelo e versões, fontes/IDs usados, custo/latência, resultado sanitizado e decisão de revisão. Conteúdo completo só para conversa visível/retida conforme política; não coletar chain-of-thought nem registrar conteúdo íntimo em logs técnicos.

## Etapas futuras propostas

1. Consolidar módulos, relações, permissões e exclusão no núcleo 2.0; concluir primeiro as funções delimitadas da [IA atual](doc:ia-e-memoria).
2. Fazer protótipo de busca em diário e histórico de veículos com dados sintéticos/selecionados, medindo busca textual, SQL e `pgvector`; aprovar quais fontes podem ser indexadas.
3. Lançar RAG somente leitura para os módulos selecionados, com fontes clicáveis, respostas com período e regra de abstenção.
4. Adicionar consultas estruturadas e ferramentas de leitura por módulo; garantir que cálculos venham do domínio.
5. Propor ações revisáveis, uma ferramenta e um módulo por vez. Avaliar assistência proativa separadamente; voz é canal opcional, não pressuposto.

Critérios de avanço, política de retenção, custos e modelo/provedor continuam pendentes. Até esses gates serem aprovados, RAG/embeddings não entram na primeira migração nem no escopo mínimo do primeiro lançamento.
