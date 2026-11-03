---
id: escopo-2-0
title: Módulos do James
description: Módulos do James preservados e evoluções planejadas com profundidade progressiva.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: financas, tags, veiculos, diario, documentos-arquivos, migrations, enums, decisoes-e-pendencias, assistente-pessoal-rag, modulos-e-integracoes, saude-treinos
source_refs:
updated: 2026-11-03
---

## Núcleo que deve migrar

[Finanças](doc:financas), [cartões](doc:cartoes), [tags](doc:tags), [acertos](doc:acertos), [pessoas/contatos](doc:pessoas-empresas), [documentos e arquivos](doc:documentos-arquivos), [notificações](doc:notificacoes) e [auditoria](doc:auditoria-2-0). Preservar funcionalidades e dados úteis do 1.x, registrando mudanças que alterem relatórios ou regras. A biblioteca visual completa pode vir depois, mas os anexos e vínculos entram na migração do núcleo.

Melhorias centrais: vários pagamentos por transação, pagamentos de fatura independentes, arquivamento de instrumentos, tags específicas em transações e itens com criação automática pela IA quando necessário, vínculos financeiros com veículos e central de atenção útil.

## Módulos planejados

| Área | Primeira fatia e vínculos |
| --- | --- |
| [Veículos](doc:veiculos) | Cadastro independente de patrimônio, abastecimento preenchido por nota fiscal, custos ligados a itens financeiros, serviço com data real, diário genérico, odômetro e checklist. |
| [Produtos e preços](doc:produtos-precos) | Catálogo pessoal por código/apresentação, preços normalizados vindos de compras e comparação por lugar; cesta “Minha inflação” como evolução posterior. |
| [Agenda e tarefas](doc:agenda-tarefas) | Compromissos, checklists, pendências e lembretes; não duplica fatos financeiros. Inclui feed `.ics` para assinatura no Calendário do iPhone; CalDAV e Google Calendar podem ser avaliados depois. |
| [Diário](doc:diario) | Texto, revisões, anexos e contexto da vida; conecta fatos reais e preserva narrativa original. |
| [Documentos e arquivos](doc:documentos-arquivos) | Biblioteca pessoal sobre o mesmo documento usado como anexo em transações, acertos, bens, veículos e diário; versões e prazos, com OCR em fase posterior. |
| [Lugares e check-ins](doc:lugares-checkins) | Check-in manual no site, com posição opcional obtida pelo navegador após ação e permissão. |
| [Patrimônio](doc:patrimonio) | Bens, avaliações e documentos sem duplicar ativos financeiros ou veículos. |
| [Saúde e treinos](doc:saude-treinos) | Rotinas, sessões e medições tipadas e privadas; importação manual de arquivo do Apple Saúde pelo navegador é candidata posterior, sujeita à validação do formato. |
| [IA](doc:ia-e-memoria) | Primeiro: classificar transações/itens com tags específicas, criar tags ausentes quando necessário e usar IA como fallback fiscal; comandos de agenda e rascunhos por texto sem nota estão em avaliação. Evolução posterior do assistente pessoal está planejada em [Memória e RAG](doc:assistente-pessoal-rag). |

Consultas opcionais da BrasilAPI complementam os formulários de empresa, endereço, instituição financeira, item fiscal, veículo e agenda. São pesquisas pontuais com resultado revisável, nunca requisito para o James funcionar; usos e limites estão nos [componentes técnicos](doc:componentes-tecnicos).

As conexões entre os módulos existentes e as ideias de novas áreas — manutenção da casa, viagens e projetos pessoais — estão em [Módulos candidatos e integrações](doc:modulos-e-integracoes). Documentos e arquivos formam um módulo planejado, com sua base de anexos no núcleo; a biblioteca completa e OCR ainda exigem priorização.

## Corte do primeiro lançamento

Essas áreas estão planejadas; todas não precisam ser construídas antes da primeira migração real. O primeiro marco utilizável deve preservar o núcleo atual com dados reconciliados. Veículos, diário e captura podem ser entregues nas próximas fatias sem quebrar o contrato de identidade e portabilidade.

Cada módulo só avança para implementação com campos, estados, ações, autorização, arquivos, auditoria, casos de erro e critérios de aceite definidos. As notas de [migrations](doc:migrations) e [enums](doc:enums) já iniciam esses contratos.

## Evoluções posteriores

Colaboração entre usuários, conta bancária conjunta, sincronização bancária e conexões com APIs móveis não são pré-requisitos do núcleo migrável. O James não terá app iOS; qualquer importação opcional do Apple Saúde depende de arquivo exportado pela pessoa e enviado no navegador, após validação do formato. O assistente pessoal do James, com RAG e ferramentas entre módulos, está planejado em fases posteriores e não é pré-requisito do primeiro lançamento. Voz e ações proativas/autônomas continuam decisões separadas; o caminho inicial é busca fundamentada somente leitura, com dados pessoais isolados.
