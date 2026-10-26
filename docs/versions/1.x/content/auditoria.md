---
id: auditoria
title: Auditoria e logs
description: Rastreabilidade das mutações de negócio, autores, filtros, retenção e diferenças de atributos.
type: architecture
status: observed
visibility: public
tags: auditoria, logs, activitylog, rastreabilidade
related: contatos, acertos, notificacoes, decisoes, arquitetura
source_refs: https://github.com/james-suite/james/blob/master/routes/web.php, https://github.com/james-suite/james/blob/master/app/Http/Controllers/AuditController.php, https://github.com/james-suite/james/blob/master/app/Enums/AuditAction.php, https://github.com/james-suite/james/blob/master/app/View/Components/ActivityLog.php, https://github.com/james-suite/james/blob/master/config/activitylog.php, https://github.com/james-suite/james/blob/master/database/migrations/2026_07_18_162729_create_activity_log_table.php
---

## O que é registrado

O painel autenticado `/audit` usa `spatie/laravel-activitylog` para registrar mutações em entidades de negócio. O log guarda o sujeito alterado, o autor, a ação, o nome do log e as mudanças de atributos.

{{diagram:audit-flow}}

O registro não é um log de todas as requisições HTTP. Ele acompanha eventos de modelos que optaram por `LogsActivity`, sempre priorizando o que mudou no dado de negócio.

## Entidades monitoradas

Hoje participam do mecanismo de auditoria:

- usuário;
- contato e grupo de contatos;
- acerto individual, divisão de conta e arquivamento de saldo;
- conta financeira;
- cartão de crédito e fatura;
- transação e item de transação;
- recorrência financeira;
- tag financeira.

O conjunto exato de eventos depende do modelo. Entidades com soft delete podem registrar exclusão lógica, restauração e exclusão permanente; entidades sem esse ciclo registram apenas criação, atualização e exclusão.

## Ações e ciclo de vida

| Descrição | Significado |
| --- | --- |
| `created` | Registro criado. |
| `updated` | Um ou mais campos foram alterados. |
| `deleted` | Registro enviado para a lixeira por soft delete. |
| `restored` | Registro restaurado. |
| `forceDeleted` | Registro removido fisicamente. |
| `item_deleted` | Evento usado para marcar a remoção definitiva de um item de transação. |

Na atualização, o log usa apenas atributos preenchíveis que ficaram sujos. Um `save()` sem alteração real não cria uma entrada vazia.

## Quem realizou a mudança

O `causer` normalmente é o usuário autenticado. Alterações disparadas por comandos do scheduler ou jobs sem sessão HTTP podem não ter `causer_id`; o painel as apresenta como **Sistema / Rotina Automática**.

Isso permite diferenciar, por exemplo:

- uma transação editada manualmente;
- uma recorrência materializada pelo scheduler;
- uma fatura avançada para o período seguinte;
- uma alteração de dados feita por um job de importação.

## Filtros do painel

Em `/audit`, a consulta é paginada em até 100 registros e pode ser filtrada por:

- **Módulo/sujeito** (`subject_type`);
- **ID do sujeito** (`subject_id`);
- **Ação** (`description`);
- **Usuário** ou opção **Sistema** para registros sem autor;
- **Data inicial** e **data final**;
- ordem mais recente ou mais antiga.

As opções de módulo, ação e usuário são montadas a partir do próprio histórico existente. Isso evita uma lista fixa que fique desatualizada quando um novo modelo passa a registrar atividades.

## Visualização do diff

Ao abrir `/audit/{activity}`, o James normaliza os dados da atividade e mostra:

- valor anterior (`old`);
- valor novo (`attributes`);
- campos criados ou removidos;
- autor, sujeito, ação, data e nome do log.

O formatador trata arrays e objetos como JSON, datas com o timezone da aplicação, valores nulos como `null` e booleanos como `true`/`false`. Quando a entidade ainda está disponível, a tela tenta criar um link para seu registro.

Em exclusões, o activity log pode guardar o estado anterior em `old` ou em `attributes`, dependendo do evento. O controller normaliza os dois formatos para que a tela mostre os dados que foram removidos.

## Histórico dentro das telas

Além do painel global, o componente `ActivityLog` aparece nas telas de entidades que oferecem histórico contextual. Ele carrega os 20 eventos mais recentes do sujeito, junto com o avatar do autor quando existe.

Assim, a investigação pode começar pelo detalhe de um contato, acerto ou entidade financeira e depois continuar no filtro global `/audit`.

## Configuração e retenção

As opções principais estão em `config/activitylog.php`:

| Configuração | Padrão | Efeito |
| --- | --- | --- |
| `ACTIVITYLOG_ENABLED` | `true` | Liga ou desliga a gravação. |
| `clean_after_days` | `365` | Idade usada pelo comando de limpeza do pacote. |
| `include_soft_deleted_subjects` | `false` | Define se a relação de sujeito inclui modelos apagados logicamente. |
| `ACTIVITYLOG_BUFFER_ENABLED` | `false` | Permite buffer de atividades para inserção em lote, quando necessário. |

O valor de 365 dias é a política padrão para uma futura execução de limpeza; não significa que cada requisição apague logs automaticamente. Se a limpeza for adotada em produção, ela deve ser executada de forma consciente porque reduz a capacidade de investigação histórica.

## Como habilitar uma nova entidade

Uma nova model de negócio deve:

1. usar `LogsActivity`;
2. declarar em `$recordEvents` somente eventos relevantes;
3. retornar `LogOptions::defaults()` com `logFillable()`;
4. usar `logOnlyDirty()` e `dontLogEmptyChanges()`;
5. escolher um nome de log específico com `useLogName()`;
6. evitar registrar segredos, tokens ou campos técnicos que não pertençam ao histórico funcional.

Exemplo mínimo:

```php
use Spatie\Activitylog\Models\Concerns\LogsActivity;
use Spatie\Activitylog\Support\LogOptions;

class ExampleModel extends Model
{
    use LogsActivity;

    protected static array $recordEvents = ['created', 'updated', 'deleted'];

    public function getActivitylogOptions(): LogOptions
    {
        return LogOptions::defaults()
            ->logFillable()
            ->logOnlyDirty()
            ->dontLogEmptyChanges()
            ->useLogName('example_model');
    }
}
```

### Caso especial: itens de transação

Itens removidos durante a edição de uma transação financeira podem ser excluídos fisicamente para não continuar compondo cálculos. Antes da remoção, o James registra o evento com descrição `item_deleted`, preservando descrição, quantidade, preço unitário, total e vínculo com a transação no histórico.

### Referências

- [Contatos](doc:contatos) e [Acertos](doc:acertos) — exemplos de telas com histórico contextual.
- [Rotinas automáticas](doc:automacoes) — explica por que o autor pode aparecer como sistema.
- [Decisões arquiteturais](doc:decisoes) — escolhas de auditoria e dados.
