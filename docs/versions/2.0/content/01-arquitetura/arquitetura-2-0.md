---
id: arquitetura-2-0
title: Arquitetura do James
description: Aplicação modular com isolamento pessoal, operações explícitas e base para memória futura.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: modelo-de-dados, componentes-tecnicos, enums, migrations, ui-ux-2-0, ia-e-memoria, assistente-pessoal-rag, decisoes-e-pendencias
source_refs:
updated: 2026-11-14
---

## Direção confirmada

Laravel com Livewire, Blade, Flux 2 gratuito e o starter kit oficial disponível quando o desenvolvimento começar em 2027. Versões de PHP, Laravel, Livewire e pacotes serão verificadas naquele momento. Flux Pro não faz parte do requisito.

O produto mantém os módulos reconhecíveis do James atual e reorganiza a persistência que limita múltiplos pagamentos, cartões e histórico. Código de domínio validado pode inspirar a implementação; a reescrita não é motivo para abandonar regras úteis.

## Implantação no servidor

Executar o James diretamente no servidor, com PHP e extensões, servidor HTTP, PostgreSQL, Meilisearch e demais serviços escolhidos instalados e gerenciados pelo sistema operacional. Workers de fila e serviços auxiliares persistentes têm processos supervisionados; o scheduler executa pelo agendador do servidor. Desenvolvimento segue instalação direta, substituindo a escolha anterior de Sail.

Manter código, configuração e dados persistentes identificados para atualização e backup. O procedimento de atualização deve prever dependências, migrations, caches e reinício dos processos persistentes. Integrações podem se comunicar por API ou processo auxiliar conforme a necessidade concreta. A decisão de implantação é operar sem Docker.

## Instalação pessoal

A instalação opera com um único usuário proprietário. A tabela `users` permanece convencional e permite múltiplos registros estruturalmente. O fluxo de instalação cria uma conta, e o produto oferece login e manutenção do próprio perfil. Cadastro público, convites, gestão de outras contas e compartilhamento ficam fora do escopo atual. O Space permanece como limite interno pessoal 1:1, vinculado ao proprietário por FK. A autenticação, as Policies e o escopo persistido continuam aplicados a todas as entradas.

Contato, empresa e participante de rateio pertencem ao catálogo do próprio usuário. Não vincular contas reais automaticamente por e-mail/documento. Dividir despesas com parcela própria zero funciona dentro do próprio histórico.

## Conta inicial e primeiro acesso

Prever um seeder específico de instalação para criar o usuário inicial e seu espaço pessoal em uma transação. Receber credenciais pela configuração protegida da instalação, usar o hasher do Laravel e manter senhas fora do código, dos logs e dos dados de exemplo. O seeder cria somente quando a instalação ainda está vazia. Reexecutar preserva a conta, suas credenciais e seus dados. Execuções concorrentes devem ser serializadas por um bloqueio de instalação.

No primeiro login, solicitar revisão de nome/e-mail e definição de senha pessoal antes de liberar os módulos. Registrar a conclusão em `onboarding_completed_at` e aplicar a restrição também nas ações do servidor. A troca de e-mail segue a verificação escolhida no starter kit. Recuperação de acesso e edição de perfil continuam disponíveis depois da configuração inicial.

O mecanismo de bootstrap mantém a cardinalidade inicial na aplicação. O banco conserva PKs e FKs convencionais, permitindo várias linhas em `users` e espaços pessoais independentes. Nunca inferir propriedade por `users.first()` ou por um ID fixo.

## Organização modular

Usar monólito Laravel modular por responsabilidades, mantendo transações locais para operações que precisam de atomicidade. Não introduzir microserviços ou motor universal de workflows. CRUD simples segue convenções do projeto; pagamentos, rateios, importações, classificações e notificações justificam Actions/Services específicas.

Fundação comum: identidade/escopo, arquivos privados, auditoria, tags, unidades/moedas, notificações, filas e portabilidade. Módulos mantêm suas tabelas e regras; relacionamentos explícitos conectam financeiro, pessoas, empresas, veículos, diário e lugares.

## Fronteiras propostas

- Documento financeiro descreve o fato; pagamento/alocação descreve a quitação; saldo não depende de tag especial.
- Cartão representa passivo com ciclos/parcelas; pagar fatura não é outra despesa.
- Rateio distingue consumidores e pagadores; nenhum acesso a outra conta é necessário.
- Catálogo de pessoas e organizações oferece identidade reutilizável para fornecedor, prestador, contato e lugar.
- Notificações têm outbox e tentativas; leitura interna não se mistura com transporte externo.
- IA é opcional e restrita à classificação de tags específicas para transações/itens e ao fallback fiscal autorizado. No modo Suggest, a pessoa aplica o resultado completo; tags ausentes são criadas na mesma operação, sem confirmação individual. O modo automático exige autorização própria. Categorias financeiras não são classificadas por IA.
- O assistente pessoal do James é uma evolução posterior: começa com busca somente leitura e respostas ligadas a fontes; ferramentas e ações chegam por módulo, com permissões e confirmações próprias. Não amplia o escopo do primeiro lançamento migrável.
- Diário e check-ins enriquecem a memória pessoal por registros no site; a localização é capturada somente em ações iniciadas pela pessoa, conforme a preferência e a permissão do navegador. Não há check-in automático ou monitoramento contínuo.

## Gates de modelagem

O [modelo de dados](doc:modelo-de-dados) propõe um registro de entidades vinculáveis e um livro financeiro interno. Validar ambos em um protótipo pequeno de pagamento misto, fatura parcial, rateio sem parcela própria e importação. Se não passarem pelo critério de simplicidade e reconciliação, simplificar antes de criar o banco definitivo.

## Contratos transversais

Servidor determina proprietário; Models, consultas, Livewire, filas, auditoria, anexos, busca e importações respeitam o mesmo escopo. Revalidar IDs e estado dentro das transações críticas. Banco garante unicidade e FKs; cache e interface são auxiliares.

Dinheiro e quantidades usam aritmética exata. Datas de negócio e instantes de captura são diferentes. Arquivamento preserva histórico; exclusão de rascunho é diferente de estorno de operação efetivada.

Number padroniza a apresentação numérica e monetária. Carbon atende datas e cálculos temporais. Locale, moeda e fuso são resolvidos pelo contexto comum, com fontes e precedência definidas em [Componentes técnicos](doc:componentes-tecnicos). Todos os módulos reutilizam esse contrato.

## Reuso entre projetos

A matriz em [Componentes técnicos](doc:componentes-tecnicos) registra os candidatos concretos do SGE e do James atual, o modo de reaproveitamento e os limites de cada peça. A regra geral é portar comportamentos com contrato útil e adaptar ao domínio pessoal do James; não importar regras acadêmicas do SGE nem carregar limitações do modelo 1.x. Só extrair uma biblioteca compartilhada quando dois projetos precisarem do mesmo contrato estável e a dependência puder ser mantida e testada sem acoplamento aos domínios.

Adotar Scout com Meilisearch para busca textual e seletores pesquisáveis, aproveitando a integração do SGE. Consultas estruturadas, integridade e cálculos continuam no PostgreSQL/Eloquent. O contrato de escopo e indexação está em [Componentes técnicos](doc:componentes-tecnicos). A [visão do assistente pessoal](doc:assistente-pessoal-rag) mantém a recuperação semântica em fase posterior, com fontes tipadas e uma fronteira pequena para provedores.

## Contratos compartilhados de integração

Donos dos fatos, publicação recuperável, revisões, idempotência, limites de somas e propagação de correções estão centralizados em [Integrações e consistência](doc:integracoes-e-consistencia). As notas de cada módulo definem somente os efeitos específicos. Módulos de agenda, tarefas e patrimônio recebem conexões opcionais por extensões, conforme a matriz de dependências.
