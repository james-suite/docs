---
id: bibliotecas-candidatas
title: Bibliotecas candidatas
description: Seleção de bibliotecas para cálculos, migração, documentos, notificações, agenda e busca.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: componentes-tecnicos, modelo-de-dados, migracao-2-0, portabilidade-backup, notificacoes, agenda-tarefas, documentos-arquivos, ia-e-memoria, assistente-pessoal-rag
source_refs: https://github.com/brick/money, https://github.com/laravel-notification-channels/webpush, https://sabre.io/vobject/, https://opis.io/json-schema/2.x/, https://laravel.com/docs/13.x/scout
updated: 2026-11-08
---

## Critério de seleção

Pesquisa realizada em 8 de novembro de 2026 a partir da documentação e dos repositórios oficiais, comparando também os manifests atuais do James e do SGE. Os encaixes abaixo são propostas para avaliar quando cada módulo entrar em desenvolvimento em 2027. A adoção depende das versões estáveis disponíveis, compatibilidade com o starter kit escolhido, licença e teste do fluxo real.

Priorizar bibliotecas que resolvem formatos, protocolos ou aritmética bem delimitados. As regras de domínio seguem em [Componentes técnicos](doc:componentes-tecnicos), e os contratos entre módulos em [Integrações e consistência](doc:integracoes-e-consistencia).

Number e Carbon integram a base de formatação e cálculos temporais do Laravel. O contexto regional compartilhado e a integração confirmada de Scout com Meilisearch estão definidos em [Componentes técnicos](doc:componentes-tecnicos). Os demais pacotes desta seleção mantêm o status de candidatos.

Antes de introduzir uma biblioteca, conferir a matriz de [Recursos do ecossistema Laravel](doc:ecossistema-laravel), que relaciona as APIs nativas aos fluxos do James e separa as ferramentas adicionais aguardando decisão.

## Aproveitar a base existente

| Biblioteca já declarada | Aplicação proposta | Integração necessária e fonte |
| --- | --- | --- |
| `spatie/laravel-medialibrary` — James e SGE, MIT | Armazenamento e conversões dos arquivos vinculados às versões de documentos. | `DocumentVersion` assume a propriedade do arquivo. Publicação, vínculos, autorização e histórico seguem o contrato de documentos. [Media Library](https://github.com/spatie/laravel-medialibrary). |
| `spatie/laravel-activitylog` — James e SGE, MIT | Registro técnico de autoria e alterações, apresentado pelo histórico contextual adaptado do SGE. | Configurar atributos permitidos, proprietário, correlação e contexto geográfico. O histórico funcional de versões permanece no domínio correspondente. [Activitylog](https://github.com/spatie/laravel-activitylog). |
| `echarts` — James, Apache-2.0 | Gráficos financeiros, evolução de preços, odômetro e medidas de saúde, conforme o módulo entregue. | Um componente compartilhado recebe séries calculadas pelo servidor, com medida, unidade, período e cobertura. [Apache ECharts](https://github.com/apache/echarts). |
| `symfony/dom-crawler` — James, MIT | Leitura determinística do HTML fiscal pelo adaptador RS já existente. | Portar parser e fixtures reais, ampliando os campos de produtos e estabelecimento conforme o novo contrato. [DomCrawler](https://github.com/symfony/dom-crawler). |
| `spatie/laravel-backup` — SGE, MIT | Backup operacional do banco e dos arquivos privados. | Adaptar configuração e procedimento de recuperação ao James. O fluxo de exportação pessoal usa seu próprio manifesto e adaptadores. [Laravel Backup](https://spatie.be/docs/laravel-backup/v10/introduction), [licença](https://github.com/spatie/laravel-backup). |

Combobox, criação inline e apresentação de auditoria têm candidatos concretos no SGE, inventariados em [Reaproveitamento de código](doc:componentes-tecnicos). Flux 2 gratuito e os componentes próprios formam a base desses seletores.

## Candidatos prioritários para os protótipos

| Biblioteca e licença | Onde ajuda | Contrato a preservar e fonte |
| --- | --- | --- |
| `brick/money` — MIT | Dinheiro exato, divisão proporcional, distribuição de centavos e arredondamento explícito em pagamentos, descontos e acertos. | Avaliar `RationalMoney` nos cálculos intermediários e `brick/math` para quantidades/preços com maior precisão. Definir moeda, escala e regra de resíduo no domínio. Persistir os valores no formato planejado. [Brick Money](https://github.com/brick/money). |
| `opis/json-schema` — Apache-2.0 | Validação estrutural do manifesto e dos registros do pacote de importação, inclusive arquivo preparado por agente externo. | Suporta JSON Schema 2020-12. Usar schemas locais versionados, com referências permitidas. Validar relações, escopo, saldos e decisões de transformação na etapa de domínio. [Opis JSON Schema](https://opis.io/json-schema/2.x/). |
| `halaxa/json-machine` — Apache-2.0 | Leitura incremental de JSON grande para staging, sem carregar a coleção inteira na memória. | Validar registros individualmente e reconciliar o conjunto no staging. Preservar decimais/IDs conforme o schema e manter anexos como arquivos separados. Uma string enorme em base64 ainda exige muita memória. [JSON Machine](https://github.com/halaxa/json-machine). |
| `league/csv` — MIT | Relatórios CSV com cabeçalhos, delimitadores, encoding e escrita incremental compartilhados. | Aplicar escape de fórmulas, declarar a transformação e conservar os valores originais no JSON. Reutilizar os mesmos normalizadores e consultas autorizadas da exportação. [League CSV](https://csv.thephpleague.com/9.0/), [escape de fórmulas](https://csv.thephpleague.com/9.0/interoperability/escape-formula-injection/), [licença](https://github.com/thephpleague/csv/blob/master/LICENSE). |
| `laravel-notification-channels/webpush` — MIT | Transporte Web Push integrado às notificações do Laravel, com VAPID e gerenciamento de assinaturas. | Adaptar a assinatura ao `NotificationEndpoint` planejado, mantendo uma única fonte de verdade. Outbox, preferências, tentativas e abertura da tela da notificação ficam coordenadas pelo James. [Laravel WebPush](https://github.com/laravel-notification-channels/webpush). |

O WebPush consultado exige Laravel 13.13 ou posterior na linha 13.x. O manifest atual do James aceita versões a partir de 13.8, então a versão efetivamente resolvida precisa ser conferida. O fluxo completo também inclui service worker, permissão e testes nos dispositivos suportados. Cada clique abre o `show` da notificação identificada no payload. [Requisitos do pacote](https://github.com/laravel-notification-channels/webpush/blob/master/composer.json).

Os protótipos financeiros usam exemplos de desconto parcial, pagamentos mistos, parcelas e acertos do [catálogo de validação](doc:cenarios-de-validacao). A biblioteca monetária fornece operações, e as regras do James determinam quais valores entram em cada cálculo.

## Agenda, mapas e documentos

| Biblioteca e licença | Aplicação proposta | Integração necessária e fonte |
| --- | --- | --- |
| `sabre/vobject` — BSD-3-Clause | Gerar e validar o feed privado `.ics`, com eventos, fusos e regras de recorrência. | Fixar UID estável, revisão e tratamento de exceções. A biblioteca cobre parte das regras de recorrência, exigindo testes dos casos aceitos e expansão limitada por período. O endpoint aplica o token revogável planejado. [VObject](https://sabre.io/vobject/), [recorrência](https://sabre.io/vobject/recurrence/), [licença](https://github.com/sabre-io/vobject/blob/master/composer.json). |
| FullCalendar Standard — MIT | Interface visual de agenda, integrada ao formulário de evento do James. | Usar os plugins gratuitos. Criar/adaptar o componente Livewire/Alpine que gerencia a instância JavaScript e envia alterações às Actions autorizadas. Conferir plugins e nomes dos pacotes na versão adotada. [FullCalendar](https://fullcalendar.io/docs), [licenciamento](https://fullcalendar.io/license). |
| `leaflet` — BSD-2-Clause | Exibir lugares, check-ins pelo site e localização associada aos registros e à auditoria. | Configurar provedor de tiles, atribuição e regras de uso. Coordenadas exibidas respeitam autorização, precisão e retenção. Revalidar a versão estável, pois a linha 2 está em beta na data da pesquisa. [Leaflet](https://leafletjs.com/), [licença](https://github.com/Leaflet/Leaflet/blob/main/LICENSE). |
| PDF.js, distribuído como `pdfjs-dist` — Apache-2.0 | Preview de PDFs dentro da biblioteca de documentos e dos anexos contextuais. | Servir os bytes por rota privada autorizada e controlar o ciclo de vida do viewer. Avaliar o custo do viewer conforme a experiência necessária. [PDF.js](https://github.com/mozilla/pdf.js). |
| `spatie/pdf-to-text` — MIT | Extrair o texto já embutido em PDFs para pesquisa, se essa função entrar no escopo. | Requer o executável `pdftotext`. PDFs digitalizados precisam de uma etapa de OCR avaliada separadamente. Executar em job com limites e conservar o original e a origem do texto derivado. [PDF to Text](https://github.com/spatie/pdf-to-text). |

## IA e busca por etapas

| Candidato | Aplicação proposta | Condição e fonte |
| --- | --- | --- |
| `laravel/ai` — MIT | SDK para os provedores de IA, saídas estruturadas e, na evolução futura, embeddings e ferramentas. | Começar nas tarefas já delimitadas, classificação de tags específicas e fallback fiscal revisável. Credenciais por usuário/instalação, orçamento e validação continuam no contrato do James. A licença do SDK e o custo do provedor têm tratamentos próprios. [Laravel AI SDK](https://laravel.com/docs/13.x/ai-sdk), [licença](https://github.com/laravel/ai). |
| `laravel/scout` — MIT, com Meilisearch, integração já declarada no SGE | Base confirmada para busca textual dos módulos e seletores pesquisáveis. | Reaproveitar a integração do SGE com escopo pessoal, indexação em fila e documentos mínimos. A evolução semântica terá avaliação própria na fase de RAG. [Scout/Meilisearch](https://laravel.com/docs/13.x/scout#meilisearch), [licença do Scout](https://github.com/laravel/scout). |

O plano de memória, fontes e permissões continua em [Assistente pessoal e RAG](doc:assistente-pessoal-rag). A busca textual usa Scout com Meilisearch. SDK, embeddings e recuperação semântica entram conforme a sequência de capacidades e são avaliados com dados e perguntas representativos do James.

## Decisão de adoção

Para cada candidato, registrar a versão escolhida, licença e dependências transitivas, requisitos de PHP/Laravel/JavaScript ou binários, responsável pelo upgrade e exemplos reais cobertos pelo protótipo. A compatibilidade precisa ser verificada com o lockfile do projeto criado em 2027.

Integrar pelas fronteiras existentes, como aritmética monetária, importador, transporte de notificação, gerador de feed e preview. Uma dependência com API pequena pode ser chamada diretamente no componente responsável. Criar adaptação compartilhada quando houver política ou conversão própria que mais de um fluxo precise reutilizar.

Validar especialmente distribuição de centavos, consumo de memória na importação, isolamento entre contas, atualização de widgets Livewire, publicação privada de documentos e restauração do backup. Para este último, configurar criptografia, recuperação de chaves e teste de restauração conforme [Portabilidade e backup](doc:portabilidade-backup). Os requisitos atuais do Backup v10 incluem PHP 8.4+, extensão ZIP e Laravel 12+, além das ferramentas de dump do banco adotado. [Requisitos operacionais](https://spatie.be/docs/laravel-backup/v10/requirements), [configuração de criptografia](https://spatie.be/docs/laravel-backup/v10/installation-and-setup).
