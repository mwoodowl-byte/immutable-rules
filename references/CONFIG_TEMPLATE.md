# CONFIG.md — Конфигурация immutable-rules

# Заполняется при установке скилла на нового агента.
# Агент задаёт вопросы владельцу и записывает ответы сюда.

## Менеджер проектов
# Где хранится PROJECTS_TASKS.md
# local = в workspace этого агента
# remote = на другом сервере (указать SSH)
pm_type: local

# Если remote — SSH доступ к серверу с менеджером проектов
# pm_ssh: sshpass -p 'PASSWORD' ssh -o StrictHostKeyChecking=no root@IP
# pm_path: /path/to/PROJECTS_TASKS.md

## GitHub
# Включить коммиты в GitHub?
github_enabled: true
# github_account: username
# Аккаунт и токен настраиваются отдельно (gh auth login)

## Оркестратор
# Telegram ID или имя оркестратора (если есть)
# orchestrator_id: 123456789
# orchestrator_name: Даня

## Владелец
# Telegram ID владельца (авторизованный отправитель)
owner_telegram_id: 309924638
owner_name: Сергей
