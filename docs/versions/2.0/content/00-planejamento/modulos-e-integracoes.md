---
id: modulos-e-integracoes
title: Módulos candidatos e integrações
description: Oportunidades de módulos futuros e fluxos que conectam os domínios do James sem duplicar dados.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento, módulos
related: planejamento-2-0, escopo-2-0, arquitetura-2-0, financas, agenda-tarefas, veiculos, patrimonio, produtos-precos, lugares-checkins, diario, saude-treinos, migration-19-health, assistente-pessoal-rag
source_refs:
updated: 2026-10-31
---

## Como avaliar uma nova área

O James já tem um conjunto amplo de módulos previstos. Uma nova área só deve ganhar tabelas e navegação próprias quando representar um fluxo recorrente com regras, histórico e consultas que não cabem bem em um módulo existente. Se for apenas uma nova forma de encontrar ou resumir registros, deve começar como integração ou visão agregada.

As propostas abaixo são **candidatas de descoberta**, não fazem parte do primeiro lançamento migrável. Elas não justificam migrations, enums ou telas definitivas antes de validar casos reais e prioridade. Os módulos existentes continuam sendo a fonte de verdade para seus fatos.

## Integrações com maior valor

| Fluxo | Integração | Regra para não duplicar fatos |
| --- | --- | --- |
| Compra com nota | Finanças ↔ empresas/lugares ↔ produtos e preços | Uma transação guarda compra e pagamento; cada item pode apontar a produto e filial. O estabelecimento da nota não prova um check-in. |
| Cartão e vencimentos | Cartões/recorrências → notificações e agenda | O módulo financeiro calcula valor, ciclo e quitação. Agenda e notificações apontam para o compromisso; não criam outro pagamento. |
| Abastecimento e manutenção | Veículos ↔ finanças ↔ produtos ↔ empresas/lugares ↔ tarefas | A compra de peça ou óleo não declara serviço concluído. O evento do veículo registra o que aconteceu; o custo continua na transação original. |
| Bens e cuidado | Patrimônio ↔ finanças ↔ tarefas/agenda ↔ pessoas/empresas ↔ documentos | Avaliação não é pagamento. Manutenção pode apontar à transação, ao prestador e ao comprovante sem replicar seus valores. |
| Registros e localização | Qualquer módulo → auditoria e contexto geográfico | Uma ação iniciada no site pode ter uma posição capturada conforme a preferência. A posição do salvamento, o endereço fiscal, o lugar e o check-in são fatos diferentes. |
| Diário e histórico | Diário ↔ eventos de outros módulos | A entrada mantém sua narrativa; links para consultas, viagens, serviços ou compromissos não copiam nem reescrevem os registros de origem. |
| Tarefas e eventos | Domínios → tarefas/agenda → notificações | Um lembrete organiza o próximo passo. Concluir a tarefa não registra abastecimento, serviço, treino, pagamento ou consulta médica como realizado. |
| Apple Health e treinos | HealthKit → saúde/treinos → diário/agenda/mapa | Importar apenas tipos autorizados. Medidas, atividade, série cardíaca e rota têm granularidades e permissões separadas; rota é parte do treino, não check-in. |
| Tags e pesquisa | Todos os módulos → tags e busca | O catálogo pode ser reutilizado, mas categoria financeira continua separada de etiqueta contextual. Sugestões de IA ficam revisáveis e vinculadas ao registro de origem. |
| Visão do dia | Finanças, agenda, tarefas, veículos e notificações → painel | O painel agrega próximos vencimentos e pendências por consulta; não vira uma segunda fonte de status. Cada ação abre o módulo dono do dado. |
| Assistente do James | Módulos autorizados → busca e comandos do assistente | Respostas apontam às fontes e consultam números nas regras do domínio. Uma proposta de escrita volta ao módulo de origem para validação e confirmação. |

Esses fluxos devem compartilhar IDs tipados, autorização por espaço pessoal, auditoria e procedência. Um vínculo facilita navegar e consultar; nunca concede acesso, substitui a regra do módulo de origem ou torna registros dependentes de uma integração externa.

## Módulos candidatos para descoberta

### Documentos, contratos e prazos

Uma área para encontrar documentos pessoais e acompanhar validade, renovação, garantia, cancelamento e reajuste de contratos ou assinaturas. Exemplos: apólice, contrato de serviço, documento de um bem, garantia de compra e prazo de renovação.

Ela reutilizaria o armazenamento privado de arquivos e apontaria para a versão canônica do documento financeiro, veículo, bem, empresa ou lugar. Um contrato poderia se relacionar a uma recorrência financeira existente e gerar lembrete de renovação; a recorrência continuaria responsável pelos valores e pagamentos. O vencimento de uma garantia não criaria despesa nem evento de manutenção.

O principal ganho seria responder “qual documento vence?”, “onde está o comprovante?” e “qual serviço/empresa está ligado a este contrato?” sem tratar anexos como uma nova cópia. Antes de planejar tabelas, definir criptografia e acesso a dados sensíveis, histórico de versões, exportação e exclusão dos arquivos.

### Casa e manutenção

Uma evolução do patrimônio e das tarefas para acompanhar imóvel, instalações e equipamentos que exigem manutenção: histórico de serviços, prestadores, garantias, leituras relevantes e próximos cuidados.

O imóvel ou equipamento teria identidade e histórico próprios. Uma manutenção real apontaria para a tarefa que a originou, o evento executado, o prestador, documentos e itens/transações financeiras relacionados. Agendamento e lembrete não comprovam execução. Começar pelos bens e veículos existentes e só separar esse módulo se houver regras específicas, como manutenção por tempo, leitura ou componente.

### Viagens e deslocamentos planejados

Uma viagem agruparia período, destinos, compromissos, reservas e referências úteis. Poderia reunir eventos da agenda, lugares visitados, transações de transporte/hospedagem, tarefas de preparação, documentos e uma entrada de diário, cada qual permanecendo no seu módulo.

O benefício seria consultar o plano e depois reconstruir o que ocorreu sem lançar a mesma reserva ou despesa duas vezes. Check-in continuaria manual no site; uma reserva ou compra não seria tratada como prova de presença. A primeira versão poderia ser apenas um agrupador de registros existentes antes de justificar um domínio de reservas próprio.

### Projetos e objetivos pessoais

Projetos com começo e resultado identificáveis — por exemplo, uma mudança, curso ou compra planejada — poderiam agrupar marcos, tarefas, eventos, documentos e despesas relevantes. Uma meta financeira ou de saúde consultaria o módulo dono dos valores e medições, sem manter cópias concorrentes.

O módulo não deve virar outro gerenciador genérico de tarefas. Sua justificativa depende de oferecer progresso e contexto de longo prazo que uma lista e tags não representem. Adiar schema até haver exemplos concretos de objetivos que cruzem mais de um domínio.

## Ordem de descoberta sugerida

1. Consolidar primeiro os vínculos e a visão agregada do dia, sem criar domínio novo.
2. Prototipar documentos e prazos porque conectam empresas, arquivos, bens, veículos, recorrências e notificações.
3. Validar manutenção da casa após o módulo de patrimônio e tarefas, reaproveitando os eventos e vínculos comprovados em veículos.
4. Tratar viagens como agrupamento de agenda, lugares, diário e finanças; só adicionar reservas próprias se o agrupamento não atender.
5. Reavaliar projetos e objetivos com casos reais; não aprovar antes de distinguir seu valor de tarefas, tags e relatórios existentes.

## Gate antes de incluir no escopo

Para cada candidato, documentar pelo menos três fluxos de uso, dono de cada dado, entidades reaproveitadas, permissões, auditoria, anexos, notificações, arquivamento/exclusão, impacto na exportação e necessidade de novas tabelas. Em seguida, decidir se é módulo, integração, consulta agregada ou se não vale entrar no James.
