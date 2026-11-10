---
id: ecossistema-laravel
title: Recursos do ecossistema Laravel
description: Recursos nativos e ferramentas próximas aplicados aos módulos, com opções adicionais separadas para decisão.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: componentes-tecnicos, bibliotecas-candidatas, arquitetura-2-0, ui-ux-2-0, decisoes-e-pendencias, fases-2-0, migracao-2-0, notificacoes
source_refs: https://laravel.com/docs/13.x, https://livewire.laravel.com/docs/4.x, https://fluxui.dev/docs/installation, https://github.com/livewire/blaze
updated: 2026-11-08
---

## Direção e alcance da pesquisa

Pesquisa da documentação oficial em 8 de novembro de 2026, usando Laravel 13, Livewire 4 e Flux 2 como referência. O desenvolvimento previsto para 2027 revalida versões e disponibilidade no starter kit escolhido. As aplicações abaixo são decisões de planejamento e precisam ser verificadas nos fluxos reais durante a implementação.

Priorizar uma API nativa que atenda ao contrato do James. Usar pacotes oficiais quando resolverem uma necessidade concreta e bibliotecas externas especializadas conforme [Bibliotecas candidatas](doc:bibliotecas-candidatas). Livewire, Flux e Blaze pertencem ao ecossistema Livewire. A procedência de cada ferramenta aparece nas referências oficiais.

## Recursos do Laravel com aplicação clara

| Recurso | Aplicação no James | Contrato de integração e fonte |
| --- | --- | --- |
| Starter kit e Fortify | Base de autenticação, recuperação de senha, confirmação e segurança da conta. | Personalizar os pontos de extensão do kit. A escolha entre registro aberto e cadastro pelo administrador permanece em pendências. [Starter kits e autenticação](https://laravel.com/docs/13.x/starter-kits#authentication). |
| Policies, Gates e validação | Propriedade dos registros, uploads privados, seleção de contatos e acesso a operações por estado. | Reutilizar regras por entrada com Form Requests ou validação Livewire. IDs de relações têm validação pelo espaço e revalidação transacional. [Autorização](https://laravel.com/docs/13.x/authorization), [validação](https://laravel.com/docs/13.x/validation). |
| Eloquent, casts e transações | Estados tipados, datas, relações e operações de pagamento, rateio e revisão. | Usar casts nativos para enums/datas e casts próprios somente onde o contrato exigir. Transações, FKs e bloqueios do banco protegem as somas e relações. [Casts](https://laravel.com/docs/13.x/eloquent-mutators), [transações](https://laravel.com/docs/13.x/database#database-transactions). |
| Number, Carbon e configuração | Formatação e contexto regional compartilhados por todos os módulos. | Seguir a fonte canônica de locale, moeda, fuso, escalas e precisão já definida em [Componentes técnicos](doc:componentes-tecnicos). |
| Collections, Lazy Collections e leitura em blocos | Normalização e geração de exportações, staging e relatórios volumosos. | Processar conjuntos com memória limitada e ordem estável. Snapshots e reconciliação permanecem responsabilidades do fluxo de portabilidade. [Lazy Collections](https://laravel.com/docs/13.x/collections#lazy-collections), [leitura em blocos](https://laravel.com/docs/13.x/eloquent#chunking-results). |
| Jobs, chains e batches | Importações, exportações, conversões documentais e indexação. | Chains representam dependências sequenciais. Batches coordenam blocos independentes e progresso quando forem úteis. A conclusão da importação depende da validação do domínio e de seus checkpoints duráveis. [Filas](https://laravel.com/docs/13.x/queues). |
| Dispatch após commit e eventos | Sincronização dos efeitos de operações confirmadas. | Despachar trabalho após commit e reutilizar a outbox planejada para intenções externas recuperáveis. Coordenar idempotência, retentativas e concorrência em [Integrações e consistência](doc:integracoes-e-consistencia). [Jobs e transações](https://laravel.com/docs/13.x/queues#jobs-and-database-transactions), [eventos](https://laravel.com/docs/13.x/events#dispatching-events-after-database-transactions). |
| Scheduler | Lembretes, recorrências, digest, limpeza de temporários e backup. | Comandos pequenos selecionam trabalho e despacham jobs. Aplicar `withoutOverlapping` e, em múltiplos servidores, `onOneServer` com cache compartilhado. Cada tarefa mantém sua idempotência persistida. [Agendamento](https://laravel.com/docs/13.x/scheduling). |
| Cache e locks | Referências públicas aceitas pela política do provedor, consultas derivadas e coordenação temporária de tarefas. | Chaves pessoais incluem espaço, revisão e parâmetros relevantes. Expiração e invalidação seguem o dono do fato. Usar `Cache::lock` onde houver exclusão temporária, com restrições definitivas no banco. [Cache](https://laravel.com/docs/13.x/cache). |
| RateLimiter e throttle | Limitar chamadas de IA, consultas externas, uploads e operações repetidas. | Definir limites por conta/função e por provedor. O orçamento financeiro da IA continua com sua reserva e reconciliação próprias. [Rate limiting](https://laravel.com/docs/13.x/rate-limiting). |
| Cliente HTTP | BrasilAPI, leitura fiscal e conectores aprovados. | Compartilhar configuração por provedor, timeouts, diagnóstico sanitizado e retentativas seletivas. `Http::pool` atende consultas independentes quando necessário. Testes usam respostas controladas e bloqueio de chamadas inesperadas. [HTTP Client](https://laravel.com/docs/13.x/http-client). |
| Storage, streams e URLs assinadas | Originais privados, staging e download de exportações. | Usar discos configurados e entrega por fluxo autorizado. URLs temporárias são capacidades de acesso limitadas no tempo, emitidas após autorização. O feed `.ics` mantém seu token revogável próprio. Media Library coordena os arquivos das versões documentais. [Storage](https://laravel.com/docs/13.x/filesystem), [URLs assinadas](https://laravel.com/docs/13.x/urls#signed-urls). |
| Notifications, Mail e locale preferido | Central interna e entrega pelos canais aprovados. | Reutilizar transporte, templates e `HasLocalePreference`. A central e a outbox mantêm seus estados planejados, com uma notificação lógica por intenção e tentativas por destino. [Notificações](https://laravel.com/docs/13.x/notifications). |
| Context e container com escopo | Correlação entre requisição, logs técnicos e jobs. | Context transporta identificadores de diagnóstico. PersonalSpaceContext deriva o proprietário autorizado e resolve preferências por execução. Payloads e logs conservam somente dados necessários, com limpeza entre execuções persistentes. [Context](https://laravel.com/docs/13.x/context), [bindings com escopo](https://laravel.com/docs/13.x/container#binding-scoped-singletons). |
| Process | Ferramentas locais de extração/conversão, quando aprovadas. | Usar API de processos com timeout, argumentos controlados e resultado sanitizado. Executar processamento pesado em job e validar os binários exigidos pelo adaptador. [Processos](https://laravel.com/docs/13.x/processes). |
| Fakes, factories e controle do relógio | Testes de pagamentos, vencimentos, importação e integrações. | Reutilizar ferramentas de teste para HTTP, filas, eventos, notificações e arquivos. Os [cenários de validação](doc:cenarios-de-validacao) definem os efeitos esperados. [Mocking](https://laravel.com/docs/13.x/mocking), [testes HTTP](https://laravel.com/docs/13.x/http-tests). |
| Health route e otimizações de deploy | Verificação de disponibilidade e preparação da aplicação. | Usar a rota de saúde e configurar os checks relevantes. Definir cache de configuração/rotas/views e reinício de workers no procedimento de atualização. [Deploy e saúde](https://laravel.com/docs/13.x/deployment). |

Scout com Meilisearch já é uma escolha confirmada, detalhada em [Componentes técnicos](doc:componentes-tecnicos). O Laravel AI SDK segue como candidato para implementar as tarefas de IA aprovadas, conforme [Bibliotecas candidatas](doc:bibliotecas-candidatas).

## Livewire e Flux nos fluxos existentes

| Recurso | Uso planejado e cuidado de integração |
| --- | --- |
| Form Objects e validação | Reutilizar campos e regras entre criação e edição quando o formulário justificar a extração. As Actions de pagamento/importação mantêm seus invariantes e autorização. [Formulários](https://livewire.laravel.com/docs/4.x/forms), [validação](https://livewire.laravel.com/docs/4.x/validation). |
| Uploads temporários | Aproveitar envio, progresso e validação antes de publicar um DocumentVersion. Configurar limite e limpeza do temporário e conservar o fluxo privado e recuperável já planejado. [Uploads](https://livewire.laravel.com/docs/4.x/uploads). |
| Computed, paginação e filtros | Evitar a repetição de consultas na mesma renderização e paginar listagens. Usar URL para filtros compartilháveis permitidos. Conteúdo sensível e rascunhos ficam no estado privado apropriado. [Computed](https://livewire.laravel.com/docs/4.x/computed-properties), [paginação](https://livewire.laravel.com/docs/4.x/pagination), [URL](https://livewire.laravel.com/docs/4.x/url). |
| Locked e autorização de actions | Proteger propriedades de identidade contra alteração pelo cliente, além de validar parâmetros e Policies em cada operação. [Segurança](https://livewire.laravel.com/docs/4.x/security). |
| Loading, dirty e offline | Dar retorno de salvamento, alterações pendentes e falha de conexão. Loading e bloqueio de botão compõem a UX junto à idempotência no servidor. [Loading](https://livewire.laravel.com/docs/4.x/loading-states), [offline](https://livewire.laravel.com/docs/4.x/wire-offline). |
| `wire:ignore` e ciclo de vida JavaScript | Integrar ECharts, calendário, mapas e PDF viewer por componentes responsáveis por inicializar, atualizar e destruir sua instância. [Integração de widgets](https://livewire.laravel.com/docs/4.x/wire-ignore), [JavaScript](https://livewire.laravel.com/docs/4.x/javascript). |
| Flux gratuito e Alpine | Usar os componentes gratuitos disponíveis para controles básicos, modais e menus. O combobox adaptado do SGE e componentes próprios atendem aos contratos de domínio. Conferir a disponibilidade gratuita de cada componente ao escolher a versão. [Flux](https://fluxui.dev/), [instalação](https://fluxui.dev/docs/installation). |

Lazy loading pode carregar painéis secundários caros sob demanda, quando medição indicar benefício. `wire:navigate` pode melhorar a navegação interna após validar widgets, listeners e navegação de retorno. A aplicação desses recursos é avaliada por tela, com preservação de foco e rascunhos. [Lazy loading](https://livewire.laravel.com/docs/4.x/lazy), [Navigate](https://livewire.laravel.com/docs/4.x/navigate).

## Ferramentas de desenvolvimento aproveitáveis

| Ferramenta | Uso concreto e fonte |
| --- | --- |
| Pint | Padronização de PHP no desenvolvimento e integração contínua, conforme as regras do projeto. [Pint](https://laravel.com/docs/13.x/pint). |
| Boost | Documentação compatível com versões e orientações para agentes de desenvolvimento. O projeto futuro deve manter seu contexto e skills atualizados conforme os pacotes usados. [Boost](https://laravel.com/docs/13.x/boost). |
| Pail | Inspeção dos logs técnicos pelo terminal durante desenvolvimento e diagnóstico. [Pail](https://github.com/laravel/pail). |
| Prompts | Interface de comandos administrativos existentes, como diagnóstico e reconstrução de índices. A migração pessoal continua pelo navegador. [Prompts](https://laravel.com/docs/13.x/prompts). |

Pint, Boost e Pail constam no manifest atual do James. Prompts faz parte da base recente do Laravel. A seleção efetiva será conferida no kit de 2027.

## Escolhas de ferramentas confirmadas

| Ferramenta | Uso e integração |
| --- | --- |
| Blaze, do ecossistema Livewire | Aproveitar a otimização de componentes Blade/Flux já incluída no starter kit Livewire consultado. Conferir a versão do kit de 2027 e compatibilidade dos componentes portados do SGE, especialmente composers, dados compartilhados e `@aware`. Estratégias adicionais de memoização e folding dependem de validação com conteúdo dinâmico. [Starter kit](https://github.com/laravel/livewire-starter-kit/blob/main/composer.json), [Blaze](https://github.com/livewire/blaze). |
| Sail | Usar como ambiente Docker de desenvolvimento, com PostgreSQL, Meilisearch e os demais serviços efetivamente escolhidos. Versionar a configuração do ambiente conforme as necessidades do James. [Sail](https://laravel.com/docs/13.x/sail). |

Na revisão com o usuário, Horizon e Pulse foram dispensados da seleção de ferramentas de produção. Pennant e Telescope foram descartados. A operação usa os recursos básicos de logs, falhas de jobs, saúde e backup previstos nos contratos do James.

## Reverb e Echo em avaliação

Reverb opera o servidor WebSocket, e Echo recebe os eventos no navegador. O benefício possível está na atualização imediata das páginas abertas durante tarefas executadas em fila ou mudanças em outra aba/dispositivo da própria conta. [Broadcasting](https://laravel.com/docs/13.x/broadcasting).

| Situação já prevista | Benefício possível |
| --- | --- |
| Importação ou exportação demorada | Atualizar progresso, conclusão e disponibilidade do arquivo na página aberta. |
| Extração fiscal ou classificação de tags por IA | Avisar quando o resultado estiver pronto para revisão, preservando o rascunho e verificando a revisão da fonte. |
| Central de notificações | Atualizar contador e lista enquanto a pessoa usa o James. Com o site fechado, os avisos seguem o fluxo Web Push planejado. |
| Registro pelo celular com painel aberto no computador | Invalidar consultas e atualizar os dados afetados, como histórico do veículo ou resumo financeiro. Um formulário em edição recebe aviso de nova revisão para a pessoa decidir como prosseguir. |

Esses são encaixes possíveis, ainda aguardando decisão. Uma implementação precisa de processo WebSocket em produção, canais privados autorizados por conta e eventos após commit. O evento pode carregar a identidade/revisão necessária para o componente recarregar os dados autorizados. [Reverb](https://laravel.com/docs/13.x/reverb), [canais privados](https://laravel.com/docs/13.x/broadcasting#authorizing-channels).

Para acompanhar uma tarefa ativa na tela, Livewire oferece consulta periódica de status com `wire:poll`. Avaliar intervalo, visibilidade do componente e encerramento da consulta quando a tarefa terminar. A decisão entre essa abordagem e eventos WebSocket depende da necessidade de atualização imediata e entre dispositivos. [Polling do Livewire](https://livewire.laravel.com/docs/4.x/wire-poll).

Octane fica para uma avaliação de desempenho se o custo de inicialização do framework aparecer nas medições. Sua adoção exige revisar contexto regional, conexões e estado por requisição em workers persistentes. A validação em tempo real dos formulários planejados usa os recursos do Livewire. Precognition pode ser reavaliado caso surja um fluxo frontend que precise de sua integração específica. [Octane](https://laravel.com/docs/13.x/octane), [Precognition](https://laravel.com/docs/13.x/precognition).

## Aplicação e revisão

Ao implementar um fluxo, consultar primeiro esta matriz e os contratos centrais, selecionando apenas as APIs necessárias. Cada integração com efeito persistente deve cumprir os testes de escopo, concorrência, retomada e precisão já definidos no planejamento. Bibliotecas e serviços adicionais aprovados entram no catálogo com versão, motivo e responsabilidade operacional.
