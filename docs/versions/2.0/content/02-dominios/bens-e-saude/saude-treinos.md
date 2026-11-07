---
id: saude-treinos
title: Saúde e treinos
description: Registros pessoais, rotinas e evolução com unidades e histórico reproduzível.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, agenda-tarefas, ia-e-memoria, migrations, migration-19-health, modulos-e-integracoes, enum-health-data-origin
source_refs: https://support.apple.com/en-euro/guide/iphone/iph5ede58c3d/ios
updated: 2026-11-07
---

## Primeira fatia

Planejar catálogo pessoal de exercícios, rotinas e sessões realizadas. A sessão congela a estrutura praticada (exercício, séries, repetições, carga, unidade e observações), para que editar uma rotina não altere os treinos antigos.

Medições corporais são observações com tipo, valor, unidade, data e fonte. Tipos e unidades pertencem a catálogos definidos; valores incompatíveis não são aceitos como JSON livre. Adicionar metas e evolução sem transformar dados anotados em diagnóstico.

## Integrações

Agenda e tarefas podem lembrar um treino; conclusão do lembrete não cria uma sessão realizada. Saúde e diário permanecem privados, fora da IA por padrão.

### Importação manual do Apple Saúde

O James não terá app iOS e não acessará diretamente o HealthKit. Se a pessoa quiser trazer dados que já estejam no app Saúde, a única integração considerada é pedir a exportação XML do próprio Apple Saúde e enviar o arquivo no navegador do James. É uma importação manual sob demanda, sem sincronização, endpoint móvel ou consulta recorrente.

Antes de definir tabelas e telas, obter um arquivo real e validar tamanho, estrutura, períodos e tipos incluídos. O export oficial é XML, mas a documentação não garante que todos os tipos, amostras detalhadas de frequência cardíaca ou rotas de treino possam ser consumidos pelo James. Não anunciar suporte a uma medida ou detalhe até confirmar isso no arquivo. Se o arquivo for grande, o upload precisa ser privado, limitado e processado em fila, com prévia e limpeza segura.

### Dados e níveis de detalhe

| Grupo | Dados que podem entrar no James | Tratamento planejado |
| --- | --- | --- |
| Medidas corporais | Peso, altura e outras medidas incluídas no arquivo | `HealthObservation` tipada, com unidade, instante/data, origem e aparelho/app de origem quando fornecidos. Valor manual e valor importado permanecem distinguíveis. |
| Atividades | Tipo, início/fim, duração, distância e energia, conforme conteúdo real do arquivo | Criar ou atualizar uma `WorkoutSession` importada; não inventar séries/repetições de musculação quando a fonte não as forneceu. |
| Movimento diário | Passos, distância caminhando/correndo e energia ativa, se presentes e úteis | Importar somente a granularidade disponível no arquivo e conservar origem; não somar amostras sobrepostas de Apple Watch, iPhone e apps terceiros. |
| Séries de atividade | Frequência cardíaca durante o treino e, quando disponíveis e desejadas, ritmo, cadência ou elevação | Guardar amostras com intervalo e unidade ligadas à sessão. Começar com resumo e gráficos sob demanda; o detalhamento ponto a ponto depende da importação manual validada e da escolha da pessoa. |
| Rota do treino | Pontos GPS ligados a um treino, se exportados em formato aproveitável | Não prometer nem persistir até validar a estrutura. Uma rota pertence àquela sessão, não ao histórico de check-ins nem ao contexto geográfico do salvamento. |
| Sono e recuperação (posterior) | Sono, frequência cardíaca de repouso e variabilidade, conforme fontes e tipos acessíveis | Adicionar apenas após definir períodos, origem, visualizações e retenção; não inferir diagnóstico ou qualidade clínica do sono. |

Não pedir acesso a todas as categorias de uma vez. Um catálogo de extensões futuras pode avaliar nutrição, respiração e outras medidas depois que houver telas e perguntas úteis para elas. Prontuários e dados clínicos não fazem parte desse escopo de atividade e exigem avaliação própria.

### Importação, origem e correções

O fluxo é iniciado pela pessoa ao enviar o XML. A tela mostra período, contagem e amostra de registros interpretados antes de confirmar. Repetir o mesmo arquivo não duplica observações; usar hash dos dados normalizados quando o arquivo não trouxer identificador estável. Uma linha manual nunca é sobrescrita por uma importação. Uma exportação posterior pode trazer dados novos ou removidos; propor duplicatas e diferenças para revisão, sem apagar valores anteriores automaticamente.

Cada amostra preserva um identificador externo estável quando disponível, nome do arquivo/lote, fonte, app/aparelho, período, unidade e instante original. Atividades sobrepostas ou repetidas no XML podem ser duplicatas: preservar a proveniência, propor correspondência e não somar energia, distância ou frequência cardíaca duas vezes. Não há conector direto do Strava previsto.

### Rotas e privacidade

Uma rota de exercício não equivale a histórico geral de localização. Importá-la permite mostrar o caminho daquele treino num mapa e correlacionar o percurso às medidas da sessão. O arquivo/linha precisa de acesso privado, retenção e exclusão próprias; nenhuma coordenada de rota deve ser copiada para `ActivityLocationContext` ou para visitas. Abrir um mapa pode revelar a região consultada ao provedor de tiles, conforme explicado em [Lugares e check-ins](doc:lugares-checkins); não enviar a trilha ao provedor de mapas.

Dados de saúde ficam fora da IA, da memória vetorial e do assistente do James por padrão. Qualquer uso futuro exige escolha específica por tipo/finalidade e uma tela que explique exatamente quais fontes e períodos serão enviados.

### Relação com os módulos existentes

- Uma atividade importada aparece no histórico de Saúde e treinos e pode ser relacionada a uma entrada de diário; o diário não copia suas amostras.
- Agenda/tarefas podem preparar um treino. O evento só vira sessão realizada quando existir registro ou importação confirmada.
- Uma rota pode relacionar a sessão a um mapa; não gera automaticamente lugar, check-in ou presença em estabelecimento.
- Resumos do dia e relatórios de saúde consultam a origem e o período. Não somar amostras sobrepostas nem tratar variação de medida como diagnóstico.

## Migração

Não há dados desse módulo no James 1.x. O primeiro desenvolvimento deve começar com seu contrato próprio, sem acrescentar campos nulos de saúde nas tabelas de contatos ou usuários. A importação XML, se aprovada após validação de arquivo, é uma extensão opcional e não requisito da migração inicial.
