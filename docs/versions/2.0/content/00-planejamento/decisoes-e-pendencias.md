---
id: decisoes-e-pendencias
title: Decisões, propostas e pendências
description: Separação entre requisitos confirmados e escolhas de modelagem ainda em revisão.
type: planning
status: proposed
visibility: public
tags: james/2.0, planejamento
related: planejamento-2-0, modelo-de-dados, fases-2-0, migracao-2-0
source_refs:
updated: 2026-10-26
---

## Confirmado pelo usuário

- Reescrita com Laravel/Livewire e Flux 2 gratuito; início em 2027 com o starter kit disponível naquele momento.
- Documentação organizada por versão no Aurelius: o planejamento dos módulos fica em `/docs/` e a documentação 1.x em `/docs/versions/1.x/`.
- Preservar o que funciona no James atual, usando o SGE como referência de planejamento, componentes, UX e auditoria.
- Migração por arquivo completo: baixar no navegador do 1.x, enviar no navegador do 2.0 e importar na própria conta autenticada.
- Vários pagamentos numa transação, arquivamento de contas/cartões e melhoria de tags, cartões e notificações.
- Rateios podem não ter parcela própria; o usuário pode pagar/organizar sem consumir. Isso não significa conta bancária conjunta.
- Não há relação entre contas de usuários prevista inicialmente. Contatos e participantes de rateio não precisam ser usuários.
- Planejar módulos existentes no roadmap, empresas/relacionamentos, check-ins, diário e veículos com vários itens de serviço/checklist.
- Fazer check-ins pelo próprio site; não criar API ou automação externa para iOS/Android. A posição atual pode ser preenchida pelo navegador mediante ação e permissão da pessoa.
- Registrar odômetro sem exigir abastecimento; usar o QR/XML da nota para identificar, quando os dados permitirem, o posto/filial e associá-lo ao abastecimento.
- Guardar produtos com códigos e histórico de preços de compras em lugares/datas diferentes, com possibilidade de uma cesta pessoal “Minha inflação”.
- Planejar um módulo amplo de estoque/despensa para alimentos e itens gerais, com mapa de locais, quantidades, lotes e reposição. A entrada pode vir de uma linha financeira, mas o vínculo e o próprio uso do módulo são opcionais.
- Reutilizar/avaliar o scraper de notas do RS existente e prever fallback por LLM quando não houver leitor por UF, sempre com autorização, revisão e preenchimento manual disponível.
- Preparar relações empresariais e referências a bases auxiliares com fonte/proveniência; dados privados de notas não alimentam base compartilhada.
- IA inicial simples para classificação; visão futura de assistente pessoal com dados rastreáveis.

## Propostas técnicas para revisar

| Proposta | Motivo e ponto de revisão |
| --- | --- |
| Documento + pagamentos + alocações | Atende múltiplas formas, datas e quitações sem duplicar transação. |
| Livro interno balanceado | Ajuda transferências, passivos de cartão e reembolsos; validar custo com protótipo antes de aprovar. |
| Espaço pessoal 1:1 com usuário | Limite consistente de dados; sem membership/colaboração agora. |
| Record registry e vínculos tipados | Conecta diário, empresas, compras e veículos; validar que não complica os Models nem duplica entidades. |
| Categoria exclusiva + tags múltiplas | Relatórios sem dupla contagem e tags transversais; confirmar UX e conversão de is_primary. |
| Combobox com nomes pendentes | Criar tag durante o formulário sem deixar órfãs ao cancelar. |
| Chave de IA pessoal + conexão da instalação opcional | Separa custos e permite serviço oferecido pelo administrador com consentimento/quota; não é decisão comercial fechada. |
| Importação inicial em domínios vazios | Reduz mistura ambígua de dois históricos; mesclagem incremental requer projeto próprio. |
| Histórico efetivado imutável | Corrigir por inversão; exceção explícita ao SoftDeletes genérico em postings/tentativas. |

## Pendências antes do schema definitivo

1. Aprovar o recorte de lançamento: quais módulos novos precisam estar no primeiro 2.0 e quais ficam em releases seguintes.
2. Definir regime dos relatórios: caixa, competência/consumo e visão por parcelas. Aprovar diferenças mensais de caixa de cartão entre datas das compras no 1.x e quitações no 2.0; saldo final igual não basta para declarar migração equivalente.
3. Confirmar o modelo de categorias, a hierarquia e a regra de herança entre transação e itens.
4. Especificar limite usado de cartão, crédito excedente, financiamento da fatura e inclusão de parcelas futuras.
5. Escolher cadastro de contas por convite/administrador ou registro aberto; nada disso permite acessar dados alheios.
6. Definir orçamento e autorização de IA por função, provedores iniciais e retenção de diagnóstico.
7. Definir limite do pacote e comportamento ao importar numa conta já preenchida; primeira proposta bloqueia mesclagem automática.
8. Definir exclusão definitiva de conta/dados e compatibilidade com auditoria funcional vitalícia.
9. Conseguir notas de exemplo e escolher a ordem de cobertura das UFs sem scraper; definir que campos da nota podem ir ao LLM, conexão autorizada, orçamento e retenção.
10. Escolher fonte, licença e política de atualização da base auxiliar de empresas; decidir quando a cobertura e qualidade permitem sugerir relações externas.
11. Aprovar a fórmula e o limite de cobertura para o índice “Minha inflação”, mantendo cesta e período-base explícitos.

## Regra editorial

Todas as notas técnicas continuam propostas. A existência de uma nota de enum ou migration não significa aprovação, implementação ou cobertura de testes. Ao aprovar uma decisão, registrar data, motivo, efeitos na migração e documentos afetados.
