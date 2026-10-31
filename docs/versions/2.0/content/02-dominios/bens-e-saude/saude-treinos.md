---
id: saude-treinos
title: Saúde e treinos
description: Registros pessoais, rotinas e evolução com unidades e histórico reproduzível.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, agenda-tarefas, ia-e-memoria, migrations, migration-19-health, modulos-e-integracoes, enum-health-sync-status, enum-health-data-origin
source_refs: https://developer.apple.com/documentation/healthkit, https://developer.apple.com/documentation/healthkit/authorizing-access-to-health-data, https://developer.apple.com/documentation/healthkit/reading-route-data, https://support.apple.com/en-euro/guide/iphone/iph5ede58c3d/ios
updated: 2026-10-31
---

## Primeira fatia

Planejar catálogo pessoal de exercícios, rotinas e sessões realizadas. A sessão congela a estrutura praticada (exercício, séries, repetições, carga, unidade e observações), para que editar uma rotina não altere os treinos antigos.

Medições corporais são observações com tipo, valor, unidade, data e fonte. Tipos e unidades pertencem a catálogos definidos; valores incompatíveis não são aceitos como JSON livre. Adicionar metas e evolução sem transformar dados anotados em diagnóstico.

## Integrações

Agenda e tarefas podem lembrar um treino; conclusão do lembrete não cria uma sessão realizada. Saúde e diário permanecem privados, fora da IA por padrão.

### Apple Health primeiro

O conector prioritário é o Apple Health por meio do HealthKit. O app Saúde reúne dados do iPhone, Apple Watch e de outros apps/aparelhos que a pessoa autorizou a escrever nele. Isso pode evitar criar um conector independente para cada balança ou app de atividade, mas só entrega ao James os dados que realmente chegaram ao HealthKit e para os quais a pessoa autorizou leitura.

O James web não consegue consultar diretamente o HealthKit. A integração completa exige um app iOS pequeno, com HealthKit habilitado, que lê os tipos autorizados no aparelho e envia ao James somente os registros selecionados. O backend teria um endpoint autenticado e limitado à importação de saúde; isso serve à sincronização deliberada de dados do HealthKit, não a check-ins, localização atual ou rastreamento contínuo. Essa ponte é uma proposta posterior e depende de confirmar que vale manter um cliente iOS além do site.

Como alternativa sem app próprio, o app Saúde permite exportar os dados de saúde e atividade em XML e a pessoa poderia enviar esse arquivo no navegador do James. É uma opção de importação manual, útil para carga inicial ou contingência, mas o tamanho, atualizações posteriores e cobertura de dados detalhados/rotas precisam ser validados antes de prometer equivalência ao HealthKit.

### Dados e níveis de detalhe

| Grupo | Dados que podem entrar no James | Tratamento planejado |
| --- | --- | --- |
| Medidas corporais | Peso, altura e outras medidas disponíveis | `HealthObservation` tipada, com unidade, instante/data, origem e aparelho/app de origem quando fornecidos. Valor manual e valor importado permanecem distinguíveis. |
| Atividades | Tipo, início/fim, duração, distância e energia, conforme atividade e fonte | Criar ou atualizar uma `WorkoutSession` importada; não inventar séries/repetições de musculação quando a fonte não as forneceu. |
| Movimento diário | Passos, distância caminhando/correndo e energia ativa, se a pessoa quiser | Preferir totais agregados pelo HealthKit para o período e conservar a fonte; não somar amostras sobrepostas de Apple Watch, iPhone e apps terceiros. |
| Séries de atividade | Frequência cardíaca durante o treino e, quando disponíveis e desejadas, ritmo, cadência ou elevação | Guardar amostras com intervalo e unidade ligadas à sessão. Começar com resumo e gráficos sob demanda; detalhamento ponto a ponto é uma opção de sincronização, não requisito para toda conta. |
| Rota do treino | Pontos GPS associados a um treino externo | Importação separada e desativada inicialmente. Uma rota pertence àquela sessão, não ao histórico de check-ins nem ao contexto geográfico do salvamento. |
| Sono e recuperação (posterior) | Sono, frequência cardíaca de repouso e variabilidade, conforme fontes e tipos acessíveis | Adicionar apenas após definir períodos, origem, visualizações e retenção; não inferir diagnóstico ou qualidade clínica do sono. |

Não pedir acesso a todas as categorias de uma vez. Um catálogo de extensões futuras pode avaliar nutrição, respiração e outras medidas depois que houver telas e perguntas úteis para elas. Prontuários e dados clínicos não fazem parte desse escopo de atividade e exigem avaliação própria.

HealthKit usa permissões por tipo e a pessoa pode revogá-las. Por proteção de privacidade, o app não consegue distinguir com segurança “leitura negada” de “não há amostras” para vários tipos; o James não pode transformar uma consulta vazia em afirmação de que o usuário não tem aquele dado. Mostrar última sincronização e contagens recebidas, sem afirmar que a leitura cobre todo o histórico. Solicitar permissão apenas quando o usuário ativar aquele grupo.

### Sincronização, origem e correções

O fluxo inicial deve ser somente leitura, iniciado pela pessoa no app iOS ou pela carga de arquivo no site. Sem sincronização de HealthKit em segundo plano no primeiro corte. A conexão guarda estado e histórico técnico mínimo; as âncoras/cursors locais de consulta permanecem protegidos no dispositivo quando possível. Repetir um lote é idempotente, alterações e remoções na fonte podem ser reconciliadas e uma linha manual nunca é sobrescrita por uma importação.

Cada amostra preserva um identificador externo estável quando disponível, fonte, app/aparelho, período, unidade e instante original. Atividades sobrepostas vindas de mais de uma origem podem ser duplicatas: preservar a proveniência, propor correspondência e não somar energia, distância ou frequência cardíaca duas vezes. Um conector direto de Strava ou de outro serviço só deve ser considerado se houver uso real e se os dados necessários não chegarem pelo HealthKit; Strava não é requisito do plano inicial.

### Rotas e privacidade

Uma rota de exercício não equivale a histórico geral de localização. Importá-la permite mostrar o caminho daquele treino num mapa e correlacionar o percurso às medidas da sessão. O arquivo/linha precisa de acesso privado, retenção e exclusão próprias; nenhuma coordenada de rota deve ser copiada para `ActivityLocationContext` ou para visitas. Abrir um mapa pode revelar a região consultada ao provedor de tiles, conforme explicado em [Lugares e check-ins](doc:lugares-checkins); não enviar a trilha ao provedor de mapas.

James não grava dados de volta no HealthKit nesta fase. Dados de saúde ficam fora da IA, da memória vetorial e do assistente do James por padrão. Qualquer uso futuro exige escolha específica por tipo/finalidade e uma tela que explique exatamente quais fontes e períodos serão enviados.

### Relação com os módulos existentes

- Uma atividade importada aparece no histórico de Saúde e treinos e pode ser relacionada a uma entrada de diário; o diário não copia suas amostras.
- Agenda/tarefas podem preparar um treino. O evento só vira sessão realizada quando existir registro ou importação confirmada.
- Uma rota pode relacionar a sessão a um mapa; não gera automaticamente lugar, check-in ou presença em estabelecimento.
- Resumos do dia e relatórios de saúde consultam a origem e o período. Não somar amostras sobrepostas nem tratar variação de medida como diagnóstico.

## Migração

Não há dados desse módulo no James 1.x. O primeiro desenvolvimento deve começar com seu contrato próprio, sem acrescentar campos nulos de saúde nas tabelas de contatos ou usuários. A ponte HealthKit e a importação XML são extensões do módulo, não um novo módulo financeiro nem requisito da migração inicial.
