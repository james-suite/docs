---
id: pessoas-empresas
title: Pessoas, empresas e relacionamentos
description: Contatos pessoais e organizações ligados a compras, serviços, lugares e interações.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, tags, financas, lugares-checkins, veiculos, migrations
source_refs: https://github.com/james-suite/james/blob/master/app/Models/Contact.php, https://brasilapi.com.br/docs
updated: 2026-10-28
---

## Base proposta

Preservar o módulo de contatos e evoluir seu núcleo para uma entidade de parte (`Party`) com tipo pessoa ou organização. Uma empresa pode ter nome conhecido, razão social opcional, documentos, telefones, e-mails, sites e estabelecimentos/lugares. Uma empresa pode operar muitos lugares/filiais; cada lugar mantém endereço e identificadores da unidade, como o CNPJ completo de uma filial. Um contato continua simples de cadastrar: nome primeiro, demais dados opcionais.

Não usar CNPJ/CPF como chave primária, nem unir cadastros de usuários diferentes por documento. Uma empresa pode ter várias filiais e marcas; a descrição no extrato pode ser um alias, não uma organização nova. Identificadores têm esquema e país; não inferir identidade apenas por nome parecido.

## Relacionamentos úteis

Vincular pessoa a empresa com tipo e vigência (ex.: prestador, funcionário ou contato comercial), empresa a estabelecimento e transação ao fornecedor. Tipos de relacionamento são catálogo configurável; podem incluir filial de, marca de, parte do grupo e fornecedora de. Não criar um enum gigante de atividades humanas.

Relações vindas de consulta pública ou base auxiliar guardam fonte, referência, data e estado de confirmação. O James pode sugerir um vínculo, mas nomes parecidos, CNPJ de matriz ou domínio de site não justificam fusão automática. Uma associação externa aceita continua distinguível da informação digitada pela pessoa e pode ser corrigida sem alterar compras históricas.

Interações registram encontro, ligação, serviço ou observação com data e vínculos. Compras e serviços de veículo já fornecem eventos concretos; não duplicar esses eventos como interações manuais obrigatórias. Vínculos permitem encontrá-los pelo contato/empresa.

## Busca e aprendizado local

Aliases de empresa facilitam reconhecer lançamentos de cartão e NFC-e. Uma regra de categorização pode apontar para empresa confirmada. O emitente fiscal e a filial são identificações distintas; quando os dados do QR/XML contêm identificador e endereço, o James tenta resolver o lugar exato. Se só houver nome da rede, não adivinhar qual loja ou posto foi visitado. O sistema sugere correspondências, mas uma fusão de partes exige revisão, mantém os IDs de origem e pode ser desfeita quando tecnicamente seguro.

### Consulta de CNPJ

Ao informar um CNPJ, oferecer uma ação para consultar a BrasilAPI e sugerir razão social, nome de uso e dados cadastrais básicos do estabelecimento. Preencher os campos como candidatos editáveis; mostrar a origem e a data da consulta. A filial identificada pelo CNPJ não deve ser fundida automaticamente com uma matriz ou outro cadastro, e dados de sócios não são necessários para esse fluxo. Se a consulta falhar ou estiver indisponível, o cadastro manual continua completo.

## Limites

Dados continuam pessoais e isolados por conta. Não relacionar automaticamente um contato com a conta de outro usuário por e-mail ou documento. Bases auxiliares de dados empresariais ficam para decisão futura sobre fontes, licença, cobertura, qualidade, atualização e custo. O modelo local já comporta referências e relações com proveniência; eventual cache compartilhado da instalação contém somente dados externos autorizados, separado dos contatos e compras privados. Nunca enriquecer uma base compartilhada com notas fiscais pessoais.

## Migração

Cada contato 1.x vira inicialmente uma pessoa, mantendo grupos, notas, imagens, chaves de origem e acertos. Converter um contato antigo para organização exige escolha explícita. Não inventar vínculos societários ou filiais usando somente o texto do contato.
