# personal_assistant

Репозиторий для версии self-hosted Telegram personal assistant / coach на базе n8n + Obsidian + OpenRouter.

## Что есть в репозитории

- `docs/n8n_telegram_coach_handoff_2026-03-25.md` — исходный handoff по серверу, инфраструктуре и текущему состоянию системы.
- `docs/n8n_full_access_orchestrator_upgrade_2026-03-26.md` — описание апгрейда оркестратора с полным read/write доступом к vault внутри `/obsidian`, текущие результаты и известные проблемы.
- `workflows/main/main_KJBp4f0aPCZWAsjV_after_full_access_orchestrator.json` — экспорт обновлённого main workflow для импорта в n8n.

## Что реализовано в апгрейде 2026-03-26

- добавлен умный orchestrator-узел, который сам решает, когда читать контекст из Obsidian;
- добавлен full access к файлам внутри `/obsidian` для чтения и записи из orchestration-слоя;
- подготовлена база для сценариев `read -> reason -> write` вместо жёсткого хардкода по нескольким интентам;
- выполнены live smoke-tests через webhook и сохранён экспорт workflow.

## Текущее состояние

Сейчас polling-поток и основной workflow запускаются, оркестратор исполняется, но end-to-end тест ещё не полностью зелёный: после исправления ошибки `URL is not defined` осталась проблема в Telegram reply node (`Bad request - please check your parameters`).

## Быстрый старт

1. Открой документ `docs/n8n_telegram_coach_handoff_2026-03-25.md`, чтобы понять текущую инфраструктуру.
2. Изучи `docs/n8n_full_access_orchestrator_upgrade_2026-03-26.md` для понимания нового orchestration-слоя.
3. Импортируй `workflows/main/main_KJBp4f0aPCZWAsjV_after_full_access_orchestrator.json` в n8n и используй его как базу для дальнейшей доработки.

## Следующий рекомендуемый шаг

После импорта workflow — починить финальный Telegram reply step и прогнать повторный live test записи заметки в Obsidian.
