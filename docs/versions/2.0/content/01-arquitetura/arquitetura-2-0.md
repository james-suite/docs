---
id: arquitetura-2-0
title: Arquitetura do James
description: Aplicação modular com isolamento pessoal, operações explícitas e base para memória futura.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: modelo-de-dados, componentes-tecnicos, enums, migrations, ui-ux-2-0, ia-e-memoria, decisoes-e-pendencias
source_refs:
updated: 2026-10-25
---

## Direção confirmada

Laravel com Livewire, Blade, Flux 2 gratuito e o starter kit oficial disponível quando o desenvolvimento começar em 2027. Versões de PHP, Laravel, Livewire e pacotes serão verificadas naquele momento. Flux Pro não faz parte do requisito.

O produto mantém os módulos reconhecíveis do James atual e reorganiza a persistência que limita múltiplos pagamentos, cartões e histórico. Código de domínio validado pode inspirar a implementação; a reescrita não é motivo para abandonar regras úteis.

## Multiusuário inicial

Cada conta tem dados pessoais isolados. Relacionamento entre contas reais não está previsto inicialmente. O Space é o limite interno pessoal 1:1, sem memberships nem compartilhamento na primeira versão.

Contato, empresa e participante de rateio pertencem ao catálogo do próprio usuário. Não vincular contas reais automaticamente por e-mail/documento. Dividir despesas com parcela própria zero funciona dentro do próprio histórico.

## Organização modular

Usar monólito Laravel modular por responsabilidades, mantendo transações locais para operações que precisam de atomicidade. Não introduzir microserviços ou motor universal de workflows. CRUD simples segue convenções do projeto; pagamentos, rateios, importações, classificações e notificações justificam Actions/Services específicas.

Fundação comum: identidade/escopo, arquivos privados, auditoria, tags, unidades/moedas, notificações, filas e portabilidade. Módulos mantêm suas tabelas e regras; relacionamentos explícitos conectam financeiro, pessoas, empresas, veículos, diário e lugares.

## Fronteiras propostas

- Documento financeiro descreve o fato; pagamento/alocação descreve a quitação; saldo não depende de tag especial.
- Cartão representa passivo com ciclos/parcelas; pagar fatura não é outra despesa.
- Rateio distingue consumidores e pagadores; nenhum acesso a outra conta é necessário.
- Catálogo de pessoas e organizações oferece identidade reutilizável para fornecedor, prestador, contato e lugar.
- Notificações têm outbox e tentativas; leitura interna não se mistura com transporte externo.
- IA é opcional e restrita a sugestões de tags para transações/itens e fallback fiscal autorizado; tags novas exigem confirmação e categorias financeiras não são classificadas por IA.
- Diário/check-ins enriquecem a memória pessoal por registros manuais e automações pontuais autorizadas.

## Gates de modelagem

O [modelo de dados](doc:modelo-de-dados) propõe um registro de entidades vinculáveis e um livro financeiro interno. Validar ambos em um protótipo pequeno de pagamento misto, fatura parcial, rateio sem parcela própria e importação. Se não passarem pelo critério de simplicidade e reconciliação, simplificar antes de criar o banco definitivo.

## Contratos transversais

Servidor determina proprietário; Models, consultas, Livewire, filas, auditoria, anexos, busca e importações respeitam o mesmo escopo. Revalidar IDs e estado dentro das transações críticas. Banco garante unicidade e FKs; cache e interface são auxiliares.

Dinheiro e quantidades usam aritmética exata. Datas de negócio e instantes de captura são diferentes. Arquivamento preserva histórico; exclusão de rascunho é diferente de estorno de operação efetivada.

## Aproveitamento do SGE

Adaptar Policies por contexto, componentes de formulário, combobox com criação, loading, histórico contextual e tentativas de entrega. O James usa propriedade pessoal, sem importar campus, vínculo institucional, perfis acadêmicos ou uma matriz de autorização desnecessária.

Projetar a busca com Eloquent e PostgreSQL antes de adotar vetores. Avaliar o SDK de IA disponível em 2027 atrás de uma fronteira pequena. Não espalhar chamadas ao modelo pelos Controllers ou hooks de Model.
