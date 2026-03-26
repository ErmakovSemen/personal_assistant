# Main orchestrator -> genspark_delegate_v1

Минимальная врезка в главный граф:

1. После `Orchestrator Agent` добавить ветку `If Meta Improvement?`.
2. Если intent относится к улучшению самого ассистента — вызывать `Execute Sub-workflow` -> `genspark_delegate_v1`.
3. После sub-workflow отправлять пользователю короткий Telegram reply по полям `status` + `summary`.

## Пример условий для ветки

`assistant_improvement`
`debug_workflow`
`prompt_update`
`architecture_patch`
`run_system_test`

## Поля для Execute Sub-workflow

```json
{
  "task_kind": "={{ $json.meta?.intent || 'assistant_improvement' }}",
  "user_request": "={{ $json.userMessage || $json.reply || '' }}",
  "context_json": "={{ JSON.stringify({ workflow_id: 'KJBp4f0aPCZWAsjV', chat_id: $json.chatId, plan: $json.plan || null }) }}",
  "success_criteria_json": "={{ JSON.stringify(['вернуть короткий статус','если были изменения — перечислить артефакты']) }}"
}
```

## Пример ответа пользователю

```text
Статус: done
Что сделано: {{ $json.summary }}
```

Если `status=needs_input`, оркестратор должен задать один уточняющий вопрос.
