---
id: modelo-de-dados
title: Modelo de dados e invariantes
description: Convenções de persistência, relações, dinheiro, exclusão e ordem dos contratos técnicos.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: migrations, enums, financas, veiculos, arquitetura-2-0, compatibilidade-legado
source_refs:
updated: 2026-10-26
---

## Estado do desenho

Schema proposto para a reescrita em 2027; ainda não há migrations PHP do 2.0. As [notas de migrations](doc:migrations) detalham campos e restrições por domínio, e os [enums](doc:enums) definem conjuntos fechados. Timestamps físicos e APIs de pacote serão escolhidos junto ao starter kit disponível.

Dois pontos pedem protótipo antes de fechar o schema: o livro financeiro interno balanceado e o registro comum `records`. A separação entre documento, pagamento e alocação já resolve o requisito funcional; o protótipo deve provar que as camadas adicionais valem seu custo e não duplicam a fonte de verdade.

## Limite pessoal

Cada `User` possui um `Space` pessoal. Esse nome é um limite interno de dados; a primeira versão não oferece workspaces compartilhados nem memberships. Contatos, empresas e participantes de rateio são dados privados do dono, sem vínculo automático com usuários reais do sistema.

Toda tabela de domínio contém `space_id` obrigatório. O servidor resolve esse valor por identidade autenticada; Jobs, arquivos, imports e respostas de IA não fornecem autoridade para mudá-lo. As relações entre dados escopados devem usar FKs compostas `(space_id, parent_id)` para pares `(space_id, id)` únicos, além das Policies. A PK continua `id` simples, compatível com Eloquent; não adotar PK composta no Model.

## Campos e tipos comuns

| Campo / tipo | Convenção proposta |
| --- | --- |
| `id` | bigint interno, PK simples. |
| `public_id` | ULID único em `records` ou recursos sem record; endereçamento, sem dispensar autorização. |
| `space_id` | FK para espaço pessoal nas tabelas de domínio; infraestrutura/global usa proprietário explícito quando cabível. |
| `record_id` | FK única somente nas entidades registradas como vinculáveis; dados permanecem na tabela tipada. |
| `created_at`, `updated_at` | Instantes UTC; exibidos com fuso do usuário. Histórico imutável pode usar apenas criação/ocorrência. |
| `occurred_on`, `due_on` | Datas locais sem horário quando o fato só é conhecido por dia. `occurred_at` e `TimePrecision` guardam instante/fuso somente quando conhecidos. |
| `amount_minor`, `currency` | bigint e código de moeda; valores positivos com direção explícita, salvo campo de saldo calculado. Escala definida pelo catálogo da moeda; BRL usa 2 casas. |
| quantidade/preço unitário | Decimal exato com escala definida; Money/Quantity calcula sem float. Total arredondado conforme moeda. |
| `archived_at` | Ocultar de operações novas sem retirar do histórico ou zerar valores. |
| `deleted_at` | Lixeira para entidades mutáveis, respeitando referências. Não retira lançamento financeiro efetivado do cálculo. |
| `revision` | Controle de concorrência e invalidação de IA/índices; fonte alterada não aceita derivado antigo. |
| JSONB | Somente payload/snapshot com schema e versão, condições permitidas e limites. Não substituir FKs ou todas as entidades por JSON genérico. |

Valores inteiros grandes devem ser serializados sem perda para JavaScript. Moedas diferentes não se somam nem são convertidas implicitamente. Um futuro câmbio exige taxa, fonte, data e arredondamento próprios.

## Relações principais

- User 1:1 Space; Space 1:N entidades de domínio.
- Party 1:N contatos/documentos e N:N grupos; relações dirigidas entre Parties e lugares da organização.
- Transação 1:N itens; transação N:N pagamentos por alocações; pagamento atribui usuário/contato/desconhecido como pagador.
- Documento fiscal 1:N tentativas de extração; leitura revisada preenche campos tipados da transação antes da confirmação.
- Cartão 1:N faturas e débitos; débito 1:N parcelas; fatura N:N pagamentos.
- Rateio 1:N participantes, cotas e contribuições; obrigação N:N liquidações.
- Record 1:1 entidade vinculável; Record N:N tags e vínculos tipados com outros Records.
- Notificação 1:N intenções de entrega; intenção 1:N tentativas.
- Conexão de IA 1:N execuções; execução gera sugestões sujeitas à revisão da fonte.
- Veículo 1:N leituras, abastecimentos e eventos; finanças N:N veículo/itens por `vehicle_financial_links`; evento real, despesa e compra destinada ao veículo não são o mesmo fato.
- Asset pode apontar para no máximo um veículo, mas veículo não depende de Asset; tarefa/checklist acompanha e não comprova serviço.

## Exclusão e histórico

A convenção atual de SoftDeletes continua para entidades mutáveis. Postings efetivados e tentativas históricas são imutáveis e precisam de exceção documentada no 2.0: correção por operação inversa, sem exclusão que reescreva saldo. Uma lixeira não equivale a arquivamento de conta/cartão.

Auditoria funcional conserva retenção vitalícia conforme a convenção atual. Segredos e texto íntimo completo não entram nesse histórico. Arquivos temporários, staging de importação, tentativas técnicas e derivados de IA recebem políticas próprias; não conservar tudo indefinidamente apenas para a ideia de “cérebro”. Exclusão definitiva de conta e tratamento dos eventos mínimos de auditoria precisam de decisão explícita antes do lançamento.

## Índices e integridade

Priorizar índices compostos de espaço com data/status usados nas consultas. Unicidade de ocorrências, chaves de idempotência, nomes normalizados e mapas de importação pertence ao banco. Operações de soma entre linhas usam transação e bloqueio; um CHECK por linha não valida sozinho a soma de um pagamento ou lote.

Criar busca textual onde necessária e medir com dados representativos. PostgreSQL continua a referência atual; PostGIS, particionamento e vetores são decisões posteriores motivadas por consultas reais e volume. Não criar tabelas de GPS bruto ou embeddings só para antecipar todo futuro possível.

## Exemplos para revisar o schema

Os [cenários de validação](doc:cenarios-de-validacao) fixam resultados de pagamentos mistos, faturas, rateios sem parcela própria, tags, IA, importação e veículos. Eles são o gate dos protótipos e a base dos testes de comportamento futuros.
