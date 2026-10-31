---
id: modulos-e-integracoes
title: Módulos candidatos e integrações
description: Oportunidades de módulos futuros e fluxos que conectam os domínios do James sem duplicar dados.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento, módulos
related: planejamento-2-0, escopo-2-0, arquitetura-2-0, financas, acertos, pessoas-empresas, agenda-tarefas, veiculos, patrimonio, produtos-precos, lugares-checkins, diario, saude-treinos, migration-19-health, assistente-pessoal-rag
source_refs: https://support.apple.com/en-euro/guide/iphone/iph5ede58c3d/ios
updated: 2026-10-31
---

## Como avaliar uma nova área

O James já tem um conjunto amplo de módulos previstos. Uma nova área só deve ganhar tabelas e navegação próprias quando representar um fluxo recorrente com regras, histórico e consultas que não cabem bem em um módulo existente. Se for apenas uma nova forma de encontrar ou resumir registros, deve começar como integração ou visão agregada.

As propostas abaixo são **candidatas de descoberta**, não fazem parte do primeiro lançamento migrável. Elas não justificam migrations, enums ou telas definitivas antes de validar casos reais e prioridade. Os módulos existentes continuam sendo a fonte de verdade para seus fatos.

## Integrações com maior valor

| Fluxo | Integração | Regra para não duplicar fatos |
| --- | --- | --- |
| Pessoas e empresas | Pessoas/empresas ↔ finanças, acertos, lugares, veículos e produtos | Um `Party` é a identidade local da pessoa ou organização. Cada vínculo informa seu papel; gráficos agrupam pelo contato/empresa relacionado sem criar tag para cada pessoa. |
| Despesa compartilhada | Finanças ↔ acertos ↔ pessoas | A transação registra a compra; participantes registram consumo; pagamentos registram quem pagou; obrigações mostram quem deve a quem. O mesmo fato pode ser consultado por pessoa sem virar outra despesa. |
| Liquidação de acerto | Acertos ↔ pagamentos ↔ contas/livro interno | A liquidação reduz obrigações. Se passar por conta pessoal, um pagamento financeiro do tipo Settlement move o caixa e zera o receber/pagar correspondente; não vira receita/despesa nem é alocado à compra original. |
| Compra de veículo compartilhada | Veículos ↔ finanças ↔ acertos ↔ contatos | Uma transação ou seus itens podem se ligar ao veículo e ao rateio. Custos por veículo e cotas por pessoa são dimensões diferentes; não duplicar compra, serviço ou pagamento. |
| Compra com nota | Finanças ↔ empresas/lugares ↔ produtos e preços | Uma transação guarda compra e pagamento; cada item pode apontar a produto e filial. O estabelecimento da nota não prova um check-in. |
| Cartão e vencimentos | Cartões/recorrências → notificações e agenda | O módulo financeiro calcula valor, ciclo e quitação. Agenda e notificações apontam para o compromisso; não criam outro pagamento. |
| Abastecimento e manutenção | Veículos ↔ finanças ↔ produtos ↔ empresas/lugares ↔ tarefas | A compra de peça ou óleo não declara serviço concluído. O evento do veículo registra o que aconteceu; o custo continua na transação original. |
| Bens e cuidado | Patrimônio ↔ finanças ↔ tarefas/agenda ↔ pessoas/empresas ↔ documentos | Avaliação não é pagamento. Manutenção pode apontar à transação, ao prestador e ao comprovante sem replicar seus valores. |
| Recorrência e serviço | Recorrências ↔ contas/cartões ↔ agenda/tarefas ↔ notificações | A ocorrência prevista pode gerar lembrete e rascunho; só a confirmação financeira cria pagamento ou compra. Pausar/arquivar o instrumento exige tratar ocorrências futuras. |
| Evento e registro pessoal | Agenda ↔ lugares/check-ins ↔ diário ↔ anexos | Um evento pode referenciar um lugar e, depois, uma nota de diário ou documento. Evento, presença e narrativa são registros distintos; nenhuma associação cria check-in automático. |
| Registros e localização | Qualquer módulo → auditoria e contexto geográfico | Uma ação iniciada no site pode ter uma posição capturada conforme a preferência. A posição do salvamento, o endereço fiscal, o lugar e o check-in são fatos diferentes. |
| Diário e histórico | Diário ↔ eventos de outros módulos | A entrada mantém sua narrativa; links para consultas, viagens, serviços ou compromissos não copiam nem reescrevem os registros de origem. |
| Tarefas e eventos | Domínios → tarefas/agenda → notificações | Um lembrete organiza o próximo passo. Concluir a tarefa não registra abastecimento, serviço, treino, pagamento ou consulta médica como realizado; uma ação explícita pode criar o evento real a partir da tarefa. |
| Saúde e treinos | Exportação XML do Apple Saúde → saúde/treinos | Avaliar importação manual pelo navegador. Sem app iOS, consulta HealthKit e sincronização automática não fazem parte do plano; cobertura de amostras e rotas precisa ser validada com um arquivo real. |
| Tags, contatos e filtros | Todos os módulos → busca e relatórios | Tags específicas classificam contexto. Contato, empresa, veículo, produto, lugar e evento são dimensões tipadas; podem aparecer como chips/filtros e agrupar gráficos sem serem cadastrados como tags. |
| Notificações úteis | Finanças, acertos, agenda, tarefas, documentos e veículos → central de atenção | Lembretes apontam para saldo, fatura, obrigação ou manutenção de origem. Resolver/adiar atua no módulo dono do dado e não transforma leitura do aviso em quitação. |
| Visão do dia e relatórios | Finanças, acertos, agenda, tarefas, veículos e notificações → consultas agregadas | Painéis mostram bases separadas (consumo, caixa, crédito a receber, dívida a pagar, vencimentos); não viram uma segunda fonte de status nem somam medidas incompatíveis. Cada ação abre o módulo dono do dado. |
| Assistente do James | Módulos autorizados → busca e comandos do assistente | Respostas apontam às fontes e consultam números nas regras do domínio. Uma proposta de escrita volta ao módulo de origem para validação e confirmação. |

Esses fluxos devem compartilhar IDs tipados, autorização por espaço pessoal, auditoria e procedência. Um vínculo facilita navegar e consultar; nunca concede acesso, substitui a regra do módulo de origem ou torna registros dependentes de uma integração externa.

### Acertos e gráficos por pessoa

Não usar a tag “reembolso” para identificar quem participou de uma despesa ou quem devolveu dinheiro. O próprio relacionamento com `Party` serve como dimensão de relatório. A origem é o participante/cota, a obrigação ou a alocação da liquidação; a pessoa não precisa ser criada novamente como tag.

Na ficha do contato ou empresa, mostrar consumo atribuído, quem pagou ao fornecedor, obrigações criadas e liquidações por período em seções separadas. Um gráfico de consumo consulta `split_shares`; quem pagou consulta `split_contributions`; valor adiantado e saldo aberto consultam obrigações e suas alocações; caixa recebido/pago consulta liquidações efetivas. Não somar esses indicadores em um total genérico: consumo, dívida e fluxo de caixa respondem a perguntas diferentes.

“Recebi R$ 40 da Ana” reduz a obrigação da Ana e, se depositado em conta pessoal, registra um movimento ligado a essa liquidação. Não cria renda nem despesa negativa. “A loja devolveu R$ 40” é uma devolução comercial ligada à compra original. Se uma empresa devolver um adiantamento, ela é a contraparte de uma obrigação; se pagar um valor sem obrigação/adiantamento anterior, registrar como receita somente quando essa for a natureza confirmada.

### Consultas cruzadas prioritárias

- **Ficha de pessoa/empresa:** linha do tempo de compras, serviços prestados, cotas de consumo, pagamentos feitos, valores a receber/pagar e anexos. Cada seção identifica o papel da contraparte; filtro por Party pode ser combinado com tags específicas sem converter o cadastro em tag.
- **Custo real de um veículo:** somar compras/serviços ligados ao veículo e comparar combustível, km, peças, mão de obra e datas. Distinguir custo comprado, serviço realizado, valor pago por mim e parcela dividida com contatos.
- **Histórico de produto e estabelecimento:** unir descrições locais da loja, identificadores, nota fiscal, preço por unidade e filial. A tela do produto mostra diferenças e cobertura; o cadastro comercial permanece ligado à empresa/lugar.
- **O que preciso fazer em seguida:** cruzar tarefa, recorrência, fatura, obrigação a pagar e manutenção prevista numa central de atenção. Cada item oferece ação no domínio correspondente e muda de estado apenas quando esse domínio confirma o resultado.
- **Linha do tempo pessoal:** diário pode apontar para evento de agenda, check-in, serviço, treino ou transação; filtros por data/lugar mostram registros relacionados sem fundi-los numa narrativa automática.
- **Saúde e rotina:** observações e treinos importados podem ser vistos junto de diário e agenda no mesmo período, mantendo origem, unidade e precisão. Importação ou proximidade temporal não prova causalidade entre treino, sono, humor ou despesa.

Essas consultas devem ser protótipos de leitura sobre os registros existentes antes de receber tabelas ou um módulo novo. Para qualquer gráfico, definir o fato contado, o papel do vínculo, a moeda e a janela de tempo; nunca somar consumo, dívida e movimento de caixa numa métrica única.

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
