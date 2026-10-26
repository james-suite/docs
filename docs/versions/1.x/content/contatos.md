---
id: contatos
title: Contatos
description: Cadastro de pessoas, grupos, dados de contato, notas, avatares privados e vínculo com acertos.
type: feature
status: observed
visibility: public
tags: contatos, crm, grupos, medialibrary
related: acertos, auditoria, traits-e-helpers, decisoes, roadmap
source_refs: https://github.com/james-suite/james/blob/master/routes/contacts.php, https://github.com/james-suite/james/blob/master/app/Models/Contact.php, https://github.com/james-suite/james/blob/master/app/Models/ContactGroup.php, https://github.com/james-suite/james/blob/master/app/Http/Controllers/ContactController.php, https://github.com/james-suite/james/blob/master/app/Http/Controllers/ContactGroupController.php, https://github.com/james-suite/james/blob/master/app/Http/Requests/StoreContactRequest.php, https://github.com/james-suite/james/blob/master/app/Traits/Searchable.php
---

## O papel do módulo

Contatos é o cadastro relacional usado pelo restante do James. Ele guarda as pessoas e entidades com quem o usuário se relaciona, sem transformar essas pessoas em contas de login. O mesmo contato pode aparecer em grupos, acertos, mensagens compartilhadas e no histórico de auditoria.

O módulo trabalha com duas entidades:

- **Contato**: nome, categoria de relacionamento, data de nascimento, telefones, e-mails, notas e avatar.
- **Grupo de contatos**: conjunto nomeado de contatos, com notas próprias, usado para organizar o cadastro e selecionar participantes de uma divisão de conta.

{{diagram:contacts-model}}

## Fluxo de uso

### Encontrar e filtrar contatos

A listagem `/contacts` combina:

- busca por nome ou notas;
- filtro por categoria de relacionamento;
- filtro por grupo;
- ordenação pelos contatos mais recentes;
- paginação de 18 resultados por página.

A busca usa `unaccent` e `ILIKE` no PostgreSQL, portanto consegue procurar sem depender de acentos. A ordenação usa similaridade para priorizar os resultados mais próximos do termo pesquisado.

### Criar e editar

Em `/contacts/create` e `/contacts/{contact}/edit`, os campos disponíveis são:

| Campo | Comportamento |
| --- | --- |
| Nome | Obrigatório, texto de até 255 caracteres. |
| Categoria | Opcional; também alimenta as opções de filtro da listagem. |
| Data de nascimento | Opcional e validada como data. |
| Telefones | Lista de pares `label` + `value`; é possível adicionar vários. |
| E-mails | Lista de pares `label` + `value`; cada valor precisa ser um e-mail válido. |
| Notas | Texto opcional renderizado como Markdown na tela de detalhes. |
| Avatar | Imagem opcional JPEG, PNG, WebP, HEIC ou HEIF, até 10 MB. |

Telefones e e-mails são mantidos como arrays JSON no registro do contato. O rótulo diferencia, por exemplo, `Celular`, `Trabalho` e `Principal` sem exigir uma tabela para cada telefone ou e-mail.

## Grupos de contatos

O CRUD de grupos fica em `/contacts/groups`:

1. Crie um grupo com nome único e, opcionalmente, notas em Markdown.
2. Selecione contatos existentes para compor o grupo.
3. Edite o grupo para substituir sua lista de contatos.
4. Abra o grupo para consultar seus participantes.

O vínculo é muitos-para-muitos. Remover um contato de um grupo não remove o contato, seus acertos nem suas notas. Os grupos também aparecem nos filtros da listagem e podem ser usados para selecionar participantes no módulo de [Acertos](doc:acertos).

## Avatares e privacidade

O avatar não é salvo como uma URL pública no contato. A implementação usa a coleção `avatar` da Spatie Media Library:

1. A imagem é processada e convertida para WebP.
2. O recorte é centralizado em 200 × 200 pixels.
3. A coleção aceita apenas um arquivo por contato.
4. O arquivo é salvo no disco privado `avatars`.
5. A rota autenticada `contacts.avatar` serve o arquivo somente quando ele existe.

O usuário pode trocar ou remover o avatar na edição. O acesso ao arquivo passa pela aplicação, em vez de expor diretamente o caminho do storage.

## Detalhes, notas e acertos

A tela `/contacts/{contact}` reúne:

- avatar, nome, categoria e data de nascimento;
- todos os telefones e e-mails rotulados;
- saldo do contato no módulo de [Acertos](doc:acertos);
- grupos associados e o modal para sincronizar os grupos;
- notas renderizadas em Markdown;
- metadados do registro e os últimos eventos de auditoria.

O contato é a referência obrigatória dos acertos individuais. Por isso, cadastrar ou corrigir uma pessoa aqui mantém o nome, avatar e categoria consistentes nas outras telas.

## Lixeira e exclusão permanente

Excluir um contato pela tela principal executa soft delete e o envia para `/contacts/trashed`. A lixeira mantém busca, filtro por categoria, paginação de até 50 itens e duas ações:

- **Restaurar**: devolve o contato à listagem normal.
- **Excluir permanentemente**: limpa a coleção de avatar e executa a exclusão física do contato.

A exclusão normal não apaga imediatamente o arquivo nem o histórico. A exclusão permanente é a operação irreversível e deve ser usada somente quando o registro não for mais necessário.

## Auditoria

Contatos e grupos registram criação, atualização e exclusão; o contato também registra restauração e exclusão permanente. O activity log grava somente campos preenchíveis que realmente mudaram. Consulte [Auditoria e logs](doc:auditoria) para entender filtros, diff e autoria das alterações.

## Rotas principais

| Caminho | Uso |
| --- | --- |
| `/contacts` | Listagem, busca e filtros. |
| `/contacts/create` | Novo contato. |
| `/contacts/{contact}` | Detalhes do contato. |
| `/contacts/{contact}/edit` | Edição. |
| `/contacts/trashed` | Lixeira. |
| `/contacts/groups` | Listagem e CRUD de grupos. |
| `/contacts/{contact}/groups/sync` | Sincronização dos grupos do contato. |

### Referências

- [Acertos](doc:acertos) — usa contatos como participantes e calcula saldos por pessoa.
- [Traits e helpers](doc:traits-e-helpers) — inclui busca e componentes compartilhados.
- [Decisões arquiteturais](doc:decisoes) — decisões sobre Media Library e notas em Markdown.
