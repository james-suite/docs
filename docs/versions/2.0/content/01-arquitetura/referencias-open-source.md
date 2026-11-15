---
id: referencias-open-source
title: Referências open source
description: Projetos públicos para estudar jornadas, domínios e integrações do James.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento, referências
related: ui-ux-2-0, bibliotecas-candidatas, arquitetura-2-0, financas, documentos-arquivos, diario, assistente-pessoal-rag
source_refs: https://github.com/phpfour/moneymint, https://github.com/actualbudget/actual, https://github.com/firefly-iii/firefly-iii, https://github.com/paperless-ngx/paperless-ngx, https://github.com/home-assistant/core, https://github.com/n8n-io/nod, https://github.com/ArthurClune/pkm-system
updated: 2026-11-15
---

## Como usar estas referências

Estudar fluxos, modelos de domínio, estados, importação/exportação e documentação dos projetos. As referências ajudam a descobrir casos que o planejamento ainda não cobre. Não implicam dependência, cópia de código ou adoção automática de funcionalidades. A aparência final do James segue Flux UI, conforme [UI e UX](doc:ui-ux-2-0).

| Projeto | Área e ideia para observar | Limite para o James |
| --- | --- | --- |
| [MoneyMint](https://github.com/phpfour/moneymint) | Aplicação Laravel/Livewire de finanças pessoais. Comparar fluxos de registro e orçamento dentro de uma stack próxima. | Usa NativePHP e Livewire 3. Referência de implementação, não arquitetura a portar. Licença MIT. |
| [Actual Budget](https://github.com/actualbudget/actual) | Orçamento por envelopes, importação e aplicação local-first. Estudar como apresenta orçamento planejado e realizado. | Modelo de orçamento próprio e implantação diferente. Não determina o modelo financeiro do James. |
| [Firefly III](https://github.com/firefly-iii/firefly-iii) | Finanças pessoais, categorias, tags, importadores e relatórios. Observar regras e cenários de reconciliação. | Produto financeiro amplo, com conceitos que devem ser comparados ao ledger e pagamentos propostos pelo James. |
| [MoneyMatter](https://github.com/letehaha/moneymatter) | Dashboard, transações, investimentos, dívidas, patrimônio e relatórios em um produto. Referência para conexões entre módulos e visualizações patrimoniais. | Integrações bancárias e MCP do projeto não são requisitos do James. Licença AGPL-3.0, revisar obrigações antes de reutilizar código. |
| [Paperless-ngx](https://github.com/paperless-ngx/paperless-ngx) | Captura documental, OCR, arquivo pesquisável, correspondentes, tipos e duplicatas. Observar proveniência e revisão de sugestões. | James já planeja uma biblioteca documental integrada aos anexos dos módulos; avaliar interoperabilidade e fluxos sem duplicar a fonte. Licença GPL-3.0. |
| [Home Assistant](https://github.com/home-assistant/core) | Integrações modulares, entidades, eventos, automações e controle local de dispositivos. Referência para a futura integração com a casa. | Estudar APIs/contratos de integração. O James não precisa recriar motor de automação residencial. Licença Apache-2.0. |
| [Nod](https://github.com/n8n-io/nod) | Assistente pessoal, regras de confiança no servidor, memória visível/editável e ações que pedem confirmação. Comparar com o plano de assistente do James. | Projeto em evolução com políticas próprias e integração de serviços. Estudar padrões de aprovação, auditoria e memória, sem assumir seu modelo técnico. |
| [PKM System](https://github.com/ArthurClune/pkm-system) | Notas diárias, links entre páginas, busca local, anexos, sincronização offline e assistente com confirmação. Referência para diário e recuperação entre registros. | Produto centrado em notas e grafos. James mantém dados financeiros e de domínio tipados como fonte factual. |
| [Overlook](https://github.com/last-refuge/overlook) | Cliente de e-mail Gmail, organização de conversas, rascunhos, fila de saída e revisão. Complementa os padrões de e-mail registrados em [UI e UX](doc:ui-ux-2-0). | Aplicativo desktop em Rust/GPUI. Estudar experiência e estados de sincronização, mantendo a interface web do James. Licença MIT. |
| [TuffUI](https://tuffui.com/components) | Componentes para fundações visuais, gráficos, tabelas, chat e propostas de ação. Ver inventário e aplicações na página de [UI e UX](doc:ui-ux-2-0). | Referência visual/interativa. Traduzir padrões para Flux UI, Livewire e Blade. |
| [EasyUI](https://easyui.site/components) | Biblioteca de componentes com exemplos de tabelas, cards, gráficos, formulários, navegação e movimento. Complementa a referência visual. | Componentes React/Tailwind. Recriar o comportamento necessário no padrão Flux UI. |

## Revisão antes de reutilizar código

Confirmar licença no arquivo do projeto e do componente específico, atividade e compatibilidade com a versão planejada. Distinguir biblioteca, serviço executável e referência conceitual. Antes de incorporar código, verificar dependências, privacidade, manutenção, acessibilidade, segurança, custos de operação e aderência aos contratos de propriedade, exportação e auditoria do James. As [bibliotecas candidatas](doc:bibliotecas-candidatas) listam dependências técnicas avaliadas separadamente.
