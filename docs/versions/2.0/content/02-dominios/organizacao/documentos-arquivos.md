---
id: documentos-arquivos
title: Documentos e arquivos
description: Biblioteca privada de documentos pessoais, versões, prazos e anexos compartilhados entre módulos.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento, documentos
related: escopo-2-0, modulos-e-integracoes, financas, acertos, patrimonio, veiculos, diario, pessoas-empresas, migration-22-documents, migration-20-imports, migration-02-records, migration-11-notifications
source_refs:
updated: 2026-11-07
---

## Objetivo

Guardar e encontrar documentos que fazem parte da vida da pessoa, estejam ou não ligados a outro registro. Exemplos: comprovantes, escrituras, certidões de nascimento, documentos de veículos e bens, contratos, garantias, apólices e notas fiscais. Uma imagem pode ser um arquivo de documento, como a foto de um recibo; organizar galerias de fotos ou reconhecer pessoas não é objetivo deste módulo.

O documento é um registro próprio do espaço pessoal. Os anexos enviados dentro de transações, acertos, veículos, bens ou diário usam o mesmo documento. Assim, a pessoa encontra o comprovante na tela de origem e também na biblioteca, sem manter cópias separadas.

## Fluxo integrado aos anexos

No James 1.x, uploads aparecem dentro de algumas telas: transações financeiras, liquidações/grupos de acerto e avatar de contato. Transações e acertos usam a coleção `attachments`, guardada no disco privado `attachments`; o componente de mídia atual permite adicionar, listar e remover esses anexos. Os avatares usam uma coleção separada. O download dos anexos passa por rota autenticada e assinada, com os tipos de registro permitidos pelo controlador. A tabela `media` identifica um único Model proprietário por linha, então o mesmo upload não é, hoje, um documento central reutilizado por vários registros.

No 2.0, manter o envio contextual onde já é útil, mas fazê-lo criar um `Document` e sua primeira versão. A pessoa também pode escolher um documento existente. A tela de origem mostra o vínculo; a biblioteca lista todos os documentos próprios, inclusive os sem vínculo. Cada arquivo enviado em lote vira um documento separado, que pode depois ser ligado a outros registros.

Exemplo: ao registrar uma compra, anexar o comprovante cria um documento ligado à transação. Ao cadastrar o serviço do veículo relacionado, selecionar o mesmo documento cria outro vínculo; não envia nem armazena os bytes de novo. A transação continua dona do total e do pagamento, o veículo do evento de serviço e o documento do arquivo e de seus metadados.

## Organização e consultas

- Biblioteca com título, nome original, tipo, data do documento, emissor/titular, validade e data de envio quando esses dados estiverem disponíveis.
- Filtros por documentos sem vínculo, por registro/módulo, pessoa/empresa, bem/veículo e vencimento próximo.
- Busca por título e metadados no começo. Busca no conteúdo via OCR é uma evolução separada.
- Pastas/coleções pessoais são opcionais e podem agrupar documentos sem mover seus arquivos nem mudar seus vínculos.
- Tags continuam no catálogo pessoal, sem tags globais ou tags documentais pré-cadastradas. Tipo de documento e relação com um módulo são campos/vínculos, não tags automáticas.
- Status de validade pode ser derivado de `expires_on` e do fuso local. Documento sem vencimento não precisa de estado artificial; arquivar não equivale a apagar.

Evitar hierarquia obrigatória como única forma de organização. A mesma escritura pode aparecer na coleção escolhida pela pessoa, na ficha do imóvel e na biblioteca sem ter várias cópias. A tela deve mostrar claramente os registros relacionados e permitir abrir cada um depois de conferir autorização.

## Tipos de documento e dados

O tipo pode começar como nome informado pela pessoa (por exemplo “escritura”, “certidão”, “comprovante” ou “garantia”), sem taxonomia global rígida. Campos comuns e opcionais:

| Campo | Uso |
| --- | --- |
| Título | Nome simples e pesquisável escolhido pela pessoa. |
| Tipo | Rótulo local opcional; não é tag. |
| Data do documento | Data de emissão ou assinatura, distinta da data de upload. |
| Titular e emissor | Relações opcionais a `Party`, escolhidas pela pessoa. Não inferir identidade por OCR. |
| Validade | Data opcional para documentos com prazo. |
| Notas | Observação curta, sem replicar o conteúdo integral em auditoria. |
| Versões | Revisões do mesmo documento com original, autor e data preservados. |
| Vínculos | Relações tipadas com transação, acerto, pessoa/empresa, bem, veículo, tarefa, evento ou entrada de diário. |

Não é necessário exigir emissor, titular ou data em anexos simples. Para um documento renovado (por exemplo, uma certidão atualizada com identidade nova), criar outro documento e relacioná-lo ao anterior; para uma nova digitalização/correção do mesmo documento, criar uma versão. Essa distinção evita que a versão nova altere a evidência histórica.

## Vencimentos e notificações

Uma data de validade pode alimentar filtros e lembretes de renovação. O módulo não renova, cancela ou altera um contrato automaticamente. A notificação persiste no centro de notificações e seu push abre primeiro a tela `show` daquele aviso; a pessoa pode então seguir uma ação para abrir o documento.

Uma garantia pode apontar ao bem ou à transação de compra e gerar aviso antes do vencimento. Isso não registra serviço ou despesa. Uma recorrência pode apontar ao contrato; alterar pagamento ou fornecedor continua sendo responsabilidade dos módulos relacionados.

## Privacidade e segurança

Documentos ficam privados por espaço e são entregues por rota autorizada, nunca por URL pública permanente. A permissão precisa ser verificada tanto na biblioteca quanto ao baixar/visualizar por um vínculo. Um link para transação ou bem não dá acesso ao documento por si só. Operadores do James não recebem acesso ao conteúdo pessoal pela interface administrativa.

Escrituras, certidões e comprovantes podem expor dados pessoais. Não incluir arquivos, conteúdo OCR, número de documento ou notas privadas em diffs de auditoria, push ou logs técnicos. OCR, indexação de texto e provedores de IA são opcionais, assíncronos e sujeitos a autorização específica; a pessoa deve saber qual documento será processado e poder corrigir/apagar o resultado derivado sem perder o original.

Validar conteúdo real, MIME, tamanho e limites comprimido/descomprimido; não confiar só na extensão do navegador. Downloads devem ter expiração/controle, arquivos temporários precisam de limpeza e exportação deve carregar originais e metadados. Backup da base não substitui backup dos bytes. Exclusão de um vínculo não apaga documento nem outras relações; remoção definitiva mostra vínculos restantes e segue retenção/lixeira definida.

## Modelo e reutilização

Modelo lógico proposto, detalhado em [Migration 22](doc:migration-22-documents):

- `documents`: registro pessoal e metadados do documento; pode existir sem vínculo a outro módulo.
- `document_versions`: revisões imutáveis, com checksum, origem, usuário que enviou e data. Cada revisão contém um original privado na biblioteca `media`.
- `media`: continua sendo o armazenamento de binários e metadados gerenciado pelo pacote de mídia, com cada objeto pertencendo à versão de um documento, não diretamente a uma transação ou veículo.
- `records`/`record_links`: reutilizar o registro comum e os vínculos tipados já planejados para ligar um documento a vários registros. Não criar um pivô especial para cada módulo.
- `Party`: emissor/titular opcional; referências são explícitas e não transformam contato em tag.

O componente reutilizável de formulário recebe um `DocumentPicker` com duas ações: enviar novo arquivo (criando documento/versão/vínculo) ou selecionar documento existente (criando apenas vínculo). A remoção no formulário elimina o vínculo daquela origem. Evidências históricas fixam uma versão em document_link_versions e podem exigir resolver referências antes de remoção definitiva. A biblioteca possui Policy própria e reutiliza o mesmo resolvedor/download seguro.

## Migração dos anexos existentes

Na exportação do James 1.x, cada linha `media` de coleção `attachments` vira um documento com uma versão inicial e um vínculo à transação, liquidação ou grupo de acerto original. Preservar nome, MIME, tamanho, hash, bytes, ordem, data conhecida, relação e lixeira quando disponível. Se o arquivo estiver ausente ou o hash não validar, mostrar o problema na prévia e no relatório; não publicar vínculo quebrado como se estivesse íntegro.

Não fundir anexos iguais automaticamente. Dois recibos idênticos em transações distintas podem ser intencionais; checksum pode indicar duplicata para revisão, mas não muda a propriedade. Avatares da coleção `avatar` não viram documentos: continuam parte do perfil. A migração de nota fiscal separa os metadados estruturados de `FiscalDocument` do arquivo original e liga ambos ao mesmo documento quando houver um arquivo de origem.

## Entrega por etapas

1. **Fundação junto aos anexos do núcleo:** documentos e versões privadas; uploads em transações/acertos criam documento; vínculos tipados; portabilidade e permissões.
2. **Biblioteca:** listar, pesquisar metadados, filtrar documentos sem vínculo, selecionar documento existente nos formulários e abrir os registros relacionados.
3. **Prazos e coleções:** datas de validade, lembretes, agrupamentos opcionais e histórico de versão mais claro.
4. **OCR e pesquisa de conteúdo, se aprovados:** fila, provedor/permissão, resultados por página/trecho, correção humana, orçamento e limpeza independente do original.

OCR e integração com IA não são requisitos da primeira fase. Primeiro garantir que o arquivo certo, o dono, os vínculos, o histórico e a exportação estejam corretos.

## Versão usada como comprovante

Ao ligar um comprovante a uma transação, acerto ou serviço, fixar a versão do arquivo usada naquele registro. Uma nova digitalização fica disponível na biblioteca e os comprovantes históricos continuam abrindo seus originais. Vínculos de consulta podem acompanhar a versão corrente, com esse comportamento visível no formulário.

Upload, publicação e exportação conservam referências às versões e ao estado dos bytes. Um arquivo pendente de publicação mostra progresso e pode ser retomado após erro. Limpeza de staging verifica a operação e os vínculos antes de remover objetos.

## Reorganização antes da migração

Na preparação dos dados do 1.x, a pessoa pode reunir anexos duplicados em um documento com múltiplos vínculos, organizar nomes/tipos e revisar relações. O plano identifica os uploads originais e a versão usada como evidência em cada destino. A conferência apresenta as fusões e o efeito sobre as contagens de arquivos/documentos.
