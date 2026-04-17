# ONBOARDING.md — Инструкция по установке нового AI агента

> Для быстрого развёртывания нового агента в инфраструктуре GelKaravan.
> Владелец: Сергей Васильевич Подмогильный (@Semen_ko, ID: 309924638)

## Шаг 1. Установка OpenClaw

```bash
# Установить OpenClaw
npm install -g openclaw
openclaw init
```

## Шаг 2. Workspace файлы

Скопировать в `~/.openclaw/workspace/`:

| Файл | Обязательный | Описание |
|------|-------------|----------|
| `AGENTS.md` | ✅ | Правила поведения агента |
| `SOUL.md` | ✅ | Личность агента (уникальна для каждого) |
| `USER.md` | ✅ | Информация о владельце |
| `IDENTITY.md` | ✅ | Имя, роль, назначение агента |
| `TOOLS.md` | ✅ | SSH доступы, сеть, модели |
| `RULES.md` | ✅ | Неизменяемые правила (из immutable-rules скилла) |
| `RULES_LOG.md` | ✅ | Лог выполнения правил |
| `MEMORY.md` | ✅ | Долгосрочная память (уникальна для каждого) |
| `HEARTBEAT.md` | ✅ | Задачи для периодических проверок |

## Шаг 3. Скилл immutable-rules (ОБЯЗАТЕЛЬНО)

```bash
# Скопировать скилл
cp -r /путь/к/immutable-rules/ ~/.openclaw/workspace/skills/immutable-rules/
```

Скилл содержит 4 критических правила:
1. **Workflow при задаче** — проект → менеджер → бэкап → коммит → память
2. **Changelog** — append-only лог всех изменений по проекту
3. **Запрет игнорировать changelog** — нельзя сказать "не делали" без проверки
4. **Диагностика поломок** — поиск по changelog + git log

## Шаг 4. Конфигурация OpenClaw

Минимальный конфиг `openclaw.json`:

```json
{
  "agents": {
    "defaults": {
      "workspace": "/home/пользователь/.openclaw/workspace",
      "llm": { "idleTimeoutSeconds": 180 }
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "ТОКЕН_БОТА",
      "allowFrom": [309924638],
      "customCommands": [
        { "command": "rule", "description": "Записать/показать неизменяемое правило" }
      ]
    }
  }
}
```

## Шаг 5. SSH доступ к VPS (для PROJECTS_TASKS.md)

Добавить в `TOOLS.md` агента:
- VPS DO: `root@45.55.225.86` / пароль в TOOLS.md главного агента
- Менеджер проектов: `/opt/openclaw_vps/workspace/PROJECTS_TASKS.md`
- Разрешения: `/opt/openclaw_vps/workspace/PROJECT_PERMISSIONS.md`

## Шаг 6. GitHub

```bash
# Установить gh CLI
# Авторизоваться токеном с правами repo
# git config user.name / user.email
```

## Шаг 7. Проверка

После установки спросить агента: "покажи правила" — должен показать RULES.md.
Дать тестовую задачу — должен пройти весь workflow по Rule #1.
