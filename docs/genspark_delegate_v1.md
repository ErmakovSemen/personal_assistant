# genspark_delegate_v1

Отдельный n8n sub-workflow для meta-задач по самому ассистенту.

## Что делает

Принимает запросы вида:
- исправить workflow;
- улучшить prompt;
- доработать архитектуру;
- проверить результат;
- подготовить патч/артефакты.

Возвращает строгий JSON:

```json
{
  "status": "done|needs_input|failed",
  "summary": "string",
  "artifacts": [{"type":"string","path":"string","description":"string"}],
  "verification": {"result":"pass|partial|fail","tests_run":["string"]},
  "next_action": "string|null"
}
```

## Входы sub-workflow

- `task_kind`
- `user_request`
- `context_json`
- `success_criteria_json`
- `model` (optional)
- `endpoint` (optional)

## Переменные окружения

- `GENSPARK_API_KEY`
- `GENSPARK_CHAT_COMPLETIONS_URL` — если не задан, workflow использует `https://api.genspark.ai/chat/completions`
- `GENSPARK_MODEL` — опционально, по умолчанию `gemini-2.5-pro`

## Как встраивать в main orchestrator

Роутить в этот sub-workflow запросы с intent вроде:
- `assistant_improvement`
- `debug_workflow`
- `prompt_update`
- `architecture_patch`
- `run_system_test`

Дальше оркестратор отправляет в sub-workflow задачу и возвращает пользователю только короткий статус-апдейт.

## Почему так

n8n рекомендует выносить повторно используемую логику в sub-workflows и вызывать её из основного графа через Execute Sub-workflow / Call n8n Workflow Tool [Source](https://docs.n8n.io/flow-logic/subworkflows/).

Для HTTP-интеграций n8n рекомендует использовать HTTP Request node с auth/header configuration, а не вшивать запросы в произвольный код [Source](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/).

## Откуда взят default endpoint

Публичной внятной документации по endpoint я не нашёл. В открытом описании VS Code extension для Genspark указан base endpoint `https://api.genspark.ai`, а пример вызова использует путь `/chat/completions` и bearer auth [Source](https://marketplace.visualstudio.com/items?itemName=Genspark.genspark).

Поэтому sub-workflow сделан конфигурируемым: endpoint можно переопределить без изменения графа.
