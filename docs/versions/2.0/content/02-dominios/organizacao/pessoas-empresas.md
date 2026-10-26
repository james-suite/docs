---
id: pessoas-empresas
title: Pessoas, empresas e relacionamentos
description: Contatos pessoais e organizações ligados a compras, serviços, lugares e interações.
type: domain-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, domínio
related: planejamento-2-0, modelo-de-dados, tags, financas, lugares-checkins, veiculos, migrations
source_refs: https://github.com/james-suite/james/blob/master/app/Models/Contact.php
updated: 2026-10-25
---

## Base proposta

Preservar o módulo de contatos e evoluir seu núcleo para uma entidade de parte (`Party`) com tipo pessoa ou organização. Uma empresa pode ter nome conhecido, razão social opcional, documentos, telefones, e-mails, sites e estabelecimentos/lugares. Um contato continua simples de cadastrar: nome primeiro, demais dados opcionais.

Não usar CNPJ/CPF como chave primária, nem unir cadastros de usuários diferentes por documento. Uma empresa pode ter várias filiais e marcas; a descrição no extrato pode ser um alias, não uma organização nova. Identificadores têm esquema e país; não inferir identidade apenas por nome parecido.

## Relacionamentos úteis

Vincular pessoa a empresa com tipo e vigência (ex.: prestador, funcionário ou contato comercial), empresa a estabelecimento e transação ao fornecedor. Tipos de relacionamento são catálogo configurável; não criar um enum gigante de atividades humanas.

Interações registram encontro, ligação, serviço ou observação com data e vínculos. Compras e serviços de veículo já fornecem eventos concretos; não duplicar esses eventos como interações manuais obrigatórias. Vínculos permitem encontrá-los pelo contato/empresa.

## Busca e aprendizado local

Aliases de empresa facilitam reconhecer lançamentos de cartão e NFC-e. Uma regra de categorização pode apontar para empresa confirmada. O sistema sugere correspondências, mas uma fusão de partes exige revisão, mantém os IDs de origem e pode ser desfeita quando tecnicamente seguro.

## Limites

Dados continuam pessoais e isolados por conta. Não relacionar automaticamente um contato com a conta de outro usuário por e-mail ou documento. Cadastro global de empresas, enriquecimento externo e importação de diretórios ficam para decisões futuras sobre fonte, qualidade e atualização.

## Migração

Cada contato 1.x vira inicialmente uma pessoa, mantendo grupos, notas, imagens, chaves de origem e acertos. Converter um contato antigo para organização exige escolha explícita. Não inventar vínculos societários ou filiais usando somente o texto do contato.
