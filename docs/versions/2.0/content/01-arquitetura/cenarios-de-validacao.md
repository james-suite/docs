---
id: cenarios-de-validacao
title: Cenários de validação do desenho
description: Exemplos numéricos e comportamentos que o schema, a migração e as interfaces devem suportar.
type: acceptance-plan
status: proposed
visibility: public
tags: james/2.0, planejamento, validação
related: planejamento-2-0, modelo-de-dados, financas, cartoes, acertos, tags, migracao-2-0, fases-2-0
source_refs:
updated: 2026-10-26
---

Estes casos são critérios para revisar o plano e, na implementação, criar testes de domínio e integração. Não representam testes já executados ou recursos implementados. Os números abaixo usam reais e nenhuma conversão de moeda.

## Financeiro e cartões

| Caso | Resultado esperado |
| --- | --- |
| Banco começa com R$ 1.000. Compra de R$ 150, sendo Pix de R$ 50 e cartão de R$ 100. | Uma transação de R$ 150; banco R$ 950; dívida do cartão R$ 100; consumo pessoal R$ 150. |
| Depois, pagar os R$ 100 da fatura com o mesmo banco. | Banco R$ 850 e dívida zero. Consumo continua R$ 150, sem outra despesa de R$ 100. |
| Fatura de R$ 500: pagamento de R$ 300 pela conta A e R$ 200 pela B, em dias diferentes. | Duas operações com datas preservadas, fatura quitada e cada conta debitada pelo seu valor. Quitar não reescreve todas as compras. |
| Pagamento de R$ 100 quita documento A de R$ 60 e B de R$ 40. Reversão técnica de R$ 20 alocada ao A. | A volta a ter R$ 20 a pagar; B continua quitado. A reversão aponta à alocação original e não pode ser aplicada de novo com a mesma chave. |
| Compra de R$ 100 no cartão com devolução comercial de R$ 20. | Compra permanece confirmada e paga ao fornecedor; documento de devolução ligado ao original reduz consumo líquido/dívida em R$ 20 conforme o fluxo. Não reabrir cobrança de R$ 20. |
| Tentar cancelar documento que ainda tem pagamento efetivo. | Recusar cancelamento. Exigir reversão técnica quando for correção; devolução comercial usa documento relacionado. |
| Arquivar conta com R$ 250 e cartão com R$ 90 de dívida. | Seletores usuais deixam de oferecê-los; histórico e patrimônio preservam valores. Ainda é possível quitar/estornar obrigações anteriores. |
| Recorrência mensal ancorada no dia 31. | Fevereiro ajusta para o último dia; março volta ao dia 31. Rodar o job duas vezes não duplica a ocorrência. |
| Transação legada de R$ 75 sem itens. | Um item de resumo identificado, total R$ 75, sem inventar produto/quantidade. Não falhar a invariante de soma nem apresentar detalhamento inexistente. |

## Rateios e classificação

| Caso | Resultado esperado |
| --- | --- |
| Usuário paga R$ 120 para três contatos; cada um consome R$ 40 e o usuário não consome. | Saída de caixa R$ 120, crédito a receber R$ 120 e despesa pessoal zero. Não dividir por quatro. |
| Um contato devolve R$ 40 desse rateio. | Caixa aumenta R$ 40, crédito cai para R$ 80 e receita pessoal permanece zero. |
| Usuário só organiza um rateio entre contatos, sem pagar nem consumir. | Rateio e saldos entre contatos são consultáveis; nenhuma obrigação ou movimentação pessoal é criada. |
| Alterar um rateio que já possui liquidação. | Propor ajuste/revisão que preserve as obrigações liquidadas; não excluir e recriar os filhos antigos. |
| Uma transação tem dois itens de R$ 60 e R$ 40, mesma tag no cabeçalho e nos itens. | Busca e relatório dessa tag somam R$ 100, sem multiplicar os valores pela quantidade de vínculos. |
| Duas abas criam “Mercado” simultaneamente, uma com caixa/espaços diferentes. | Uma tag canônica, seleções apontando ao mesmo ID, sem erro bruto de unicidade. Cancelar um formulário não deixa tag nova órfã. |
| Tag canônica de uma pessoa coincide com alias de outra tag dela. | O namespace único encontra a tag existente ou exige resolução; não manter duas classificações ambíguas. |

## IA e notificações

| Caso | Resultado esperado |
| --- | --- |
| Resposta de IA chega depois de o usuário editar a transação. | Sugestão desatualizada, nenhuma aplicação silenciosa sobre a nova revisão. |
| Modelo devolve tag de outro espaço, ID inexistente ou instrução para mover dinheiro. | Resultado recusado; nenhuma alteração financeira ou acesso cruzado. |
| Chave pessoal falha e existe chave da instalação. | Falha identificada sem trocar de chave/provedor automaticamente. Salvar sem IA continua disponível. |
| Job de vencimento é repetido e o Telegram falha uma vez. | Uma notificação, uma intenção de entrega e tentativas distintas. Não duplicar o aviso por reenvio do transporte. |
| Fatura é paga antes do lembrete agendado. | Lembrete pendente é resolvido/cancelado; aviso antigo leva ao estado atual e não oferece cobrança já concluída. |

## Portabilidade e isolamento

| Caso | Resultado esperado |
| --- | --- |
| Usuário A envia pacote cujo usuário antigo tem o mesmo ID numérico de B no 2.0. | Dados importados somente para A; ID de origem não autoriza consulta ou gravação em B. |
| Reenviar o mesmo arquivo ou iniciar duas confirmações. | Reconhecimento de origem e exclusão mútua da importação; nenhuma duplicata. |
| Depois da prévia de destino vazio, criar uma transação antes de confirmar. | Prévia expira ou confirmação é bloqueada; nenhum dado pessoal novo é apagado para abrir espaço. |
| Uma linha antiga vira documento, pagamento e alocação; duas linhas de transferência viram uma operação. | Mapas suportam 1:N e N:1 por papel de conversão, mantendo referências e idempotência. |
| Compra em junho quitada em julho; o 1.x alterou o saldo na data da compra. | Conferir série mensal, mostrar diferença de regra temporal e preservar datas/proveniência. Saldo final igual não mascara mudança de relatório. |
| Pacote contém arquivo ausente, hash inválido, caminho ../ ou volume expandido além do limite. | Validação bloqueia a publicação e fornece diagnóstico; nenhum arquivo sai do staging privado. |

## Veículos, diário e contexto

| Caso | Resultado esperado |
| --- | --- |
| Cadastrar um veículo e não criar um bem patrimonial. | Abastecimentos, serviços, custos e diário funcionam; ausência de bem não bloqueia o módulo. |
| Vincular esse veículo a um bem patrimonial e registrar uma avaliação. | O valor aparece uma única vez no patrimônio; o vínculo não cria custo, compra ou movimento de caixa. |
| Ler nota com um item de combustível, quantidade 40 L, preço R$ 6/L e desconto R$ 4. | Rascunho fiscal e abastecimento ligados; preço líquido R$ 5,90/L, transação conciliada e pagamento ainda depende de confirmar quem pagou. |
| Ler nota com combustível e outros itens, com desconto geral no total. | O scanner não atribui o desconto inteiro ao combustível nem inventa rateio; pede alocação ou deixa o preço líquido por litro sem confirmação. |
| Confirmar nota de abastecimento paga por um contato. | Custo e pagador ficam registrados; nenhuma conta bancária pessoal é debitada e nenhuma dívida é criada sem acerto explícito. |
| Comprar óleo para um veículo em uma data e trocar semanas depois. | Compra ligada ao veículo sem serviço concluído na primeira data; evento de troca usa a data/quilometragem real e pode referenciar o item comprado. |
| Registrar “ouvi um ruído ao frear” no diário do veículo. | Nota aparece na linha do tempo, sem criar despesa, manutenção concluída ou mudança de odômetro. |
| Oficina realiza óleo R$ 80 e mão de obra R$ 40 em uma visita. | Um evento com dois itens e despesa total R$ 120 ligada a uma transação, sem gerar R$ 240 ao consultar pelos dois módulos. |
| Tanque cheio no km 1.000; parcial de 5 L no km 1.100; cheio com 15 L no km 1.250. | Intervalo completo considera 250 km e 20 L, resultando 12,5 km/L; não usar só os últimos 15 L nem o volume do abastecimento inicial. |
| Marcar checklist de manutenção concluído sem registrar o serviço. | Checklist muda de estado, mas odômetro/base do plano não são atualizados como se o serviço tivesse ocorrido. |
| Compra numa empresa com endereço conhecido. | Pode sugerir associação ao lugar; não criar visita confirmada automaticamente. |
| Editar texto íntimo do diário. | Revisão protegida no diário; Activity Log registra autoria/evento sem copiar todo o texto para diffs comuns. |

## Gate de aprovação

O protótipo de schema precisa resolver os casos centrais de dinheiro, cartão, rateio e migração antes de consolidar ledger/records. Os demais casos orientam cada marco; não exigir todos os módulos futuros para validar a migração do núcleo atual.
