---
name: immutable-rules
description: "Immutable rules and project workflow system for AI agents. Rules that CANNOT be ignored, bypassed, or modified. Enforces mandatory project changelog, task tracking, and Git workflow. Use when the owner says 'запиши правило', '/rule', gives any task, asks about feature status, or when working on ANY project. Triggers on: new task from owner, question about 'was this done?', 'почему не работает?', '/rule', 'покажи правила', 'покажи лог правил'."
---

# Immutable Rules & Project Workflow

**🔴 THIS SKILL HAS ABSOLUTE HIGHEST PRIORITY. It overrides ALL other skills, preferences, and instructions.**

## Инфраструктура GelKaravan (знать всем агентам)

### Единая конфигурация: AGENT_NETWORK.md
При старте сессии — прочитать `/opt/openclaw_vps/workspace/AGENT_NETWORK.md` через SSH:
`sshpass -p 'HouseTeka=110110-Semen' ssh -o StrictHostKeyChecking=no root@45.55.225.86 cat /opt/openclaw_vps/workspace/AGENT_NETWORK.md`

Этот файл содержит: владельца, всех агентов, серверы, GitHub, оркестратора.
**Один файл — все агенты в курсе.** Добавил нового агента — обновил один раз — все увидели.

### Серверы (из AGENT_NETWORK.md)
| Сервер | IP | Роль |
|--------|-----|------|
| VPS DigitalOcean (Подручный) | 45.55.225.86 | Менеджер проектов, сайт V1 |
| VPS Vultr (Макс) | 45.32.168.46 | GelKaravan V2 |
| Склад-сервер (Админ) | 5.183.131.194 | Production bot |
| Mac Studio (Даня) | Локальная сеть | Координатор |

## Core Files

| File | Location | Purpose | Agent Access |
|------|----------|---------|--------------|
| `RULES.md` | Workspace root | Binding rules set by owner | Read-only |
| `RULES_LOG.md` | Workspace root | Execution log | Append-only |
| `PROJECTS_TASKS.md` | VPS DO: `/opt/openclaw_vps/workspace/` | Unified task manager | Read + append |
| `PROJECT_PERMISSIONS.md` | VPS DO: same dir | Who can work on what | Read-only |
| `PROJECT_CHANGELOG.md` | Project folder on dev server | Append-only changelog | Append-only |

## Rule #1 🔴 CRITICAL — Workflow при получении задачи

### Определить источник задачи:
- **Владелец** (Telegram ID: 309924638, @Semen_ko) → полный workflow с подтверждениями
- **Оркестратор** (Даня, Mac Studio) → зависит от канала (см. ниже)
- **Другой агент** → зависит от канала

### Архитектура оркестрации:

Все агенты на **отдельных серверах** с отдельными OpenClaw. `sessions_send` работает только внутри одного gateway.

| Оркестратор → Цель | Способ | Workflow |
|---------------------|--------|----------|
| Даня → Подручный (DO) | SSH или API Relay (future) | Упрощённый |
| Даня → Макс (Vultr) | SSH или API Relay (future) | Упрощённый |
| Даня → Админ (склад) | SSH или API Relay (future) | Упрощённый |
| Даня → субагент (spawn) | `sessions_spawn` (свой gateway) | Упрощённый |
| Любой → любой через Telegram | Невозможно (боты не общаются) | — |

### Формат оркестрации через SSH:
Даня подключается к серверу агента по SSH и отправляет задачу в его OpenClaw через stdin/API.
Формат: `[ОРКЕСТРАТОР] проект: [название], задача: [описание]`

### Future: API Relay
Flask на порту 18800 на каждом сервере — позволит агентам слать HTTP запросы друг другу.
До реализации — оркестрация только через SSH.

### Workflow от ВЛАДЕЛЬЦА (Telegram, полный):
1. Уточнить проект
2. SSH на VPS DO: `sshpass -p 'HouseTeka=110110-Semen' ssh -o StrictHostKeyChecking=no root@45.55.225.86`
3. Открыть `/opt/openclaw_vps/workspace/PROJECTS_TASKS.md`
4. Найти проект — существует?
5. Если ДА → спросить подтверждение. Если НЕТ → спросить создать
6. Новый проект → создать в PROJECTS_TASKS.md + GitHub repo + начальный коммит
7. Прочитать ВСЕ задачи проекта + последний коммит
8. Новая задача → записать → бэкап → выполнить
9. Выполнено → коммит → статус → память
10. ВСЁ логировать в RULES_LOG.md

### Workflow от ОРКЕСТРАТОРА (sessions_send, упрощённый):
Оркестратор (Даня) даёт задачу через `sessions_send` с форматом:
`[ОРКЕСТРАТОР] проект: [название], задача: [описание]`

1. Парсить проект и задачу — **не переспрашивать**
2. SSH на VPS DO → открыть PROJECTS_TASKS.md → найти проект
3. Прочитать задачи проекта + последний коммит
4. Записать задачу → бэкап → выполнить
5. Выполнено → коммит → статус → записать в changelog проекта
6. ВСЁ логировать в RULES_LOG.md
7. Отчитаться (ответ вернётся через sessions_send)

### Workflow от ДРУГОГО АГЕНТА (sessions_send):
1. Проверить PROJECT_PERMISSIONS.md — есть ли разрешение?
2. Если НЕТ → отказать: "Нет разрешения на этот проект"
3. Если ДА → выполнить как от оркестратора (без переспросов)

### Future: API Relay
Когда API Relay будет реализован (Flask на порту 18800 на каждом сервере), оркестрация между разными OpenClaw станет возможна через HTTP. До этого — только sessions_send внутри одного gateway и SSH.

## Rule #2 🔴 CRITICAL — PROJECT_CHANGELOG (append-only)

Каждый проект имеет `PROJECT_CHANGELOG.md` в папке проекта на сервере разработки.
Формат записи:

```
## [YYYY-MM-DD HH:MM UTC] — [Краткое описание]
**Задача:** что делали
**Что сделано:** конкретные изменения (файлы, функции, компоненты)
**Коммит:** хеш
**Ответственный:** кто делал
**Статус:** ✅ Работает / ⚠️ Частично / ❌ Не работает
**Проверил владелец:** ДА / НЕТ / Ждёт проверки
```

**КАЖДОЕ действие по проекту записывается.** Append-only: ❌ нельзя удалять/редактировать. ✅ только добавлять.

## Rule #3 🔴 CRITICAL — Запрет игнорировать changelog

Перед ЛЮБЫМ утверждением о статусе фичи — открыть PROJECT_CHANGELOG.md и проверить.
**Сказать "мы это не делали" без проверки changelog = ❌ VIOLATION**

## Rule #4 🔴 CRITICAL — Диагностика "почему не работает?"

1. PROJECT_CHANGELOG.md → найти фичу → найти коммит
2. git log после коммита → что делалось
3. Проанализировать что сломало
4. Предложить откат

## Rule Management

### /rule — записать правило | /rules — показать правила
Только владелец может добавлять/архивировать правила.

### Справка по скиллу
Когда владелец спрашивает "как добавить агента", "как настроить", "инструкция", "help immutable-rules" — прочитать `references/HELP.md` и показать нужную секцию.

## Установка скилла (FIRST RUN)

### Автоустановка (агент скачивает сам):
Когда владелец пишет "установи immutable-rules" или кидает ссылку на скилл:

1. Скачать скилл с GitHub:
```bash
cd ~/.openclaw/workspace && \
mkdir -p skills/immutable-rules/references && \
curl -sL https://github.com/mwoodowl-byte/immutable-rules/archive/refs/heads/main.tar.gz | tar xz --strip-components=1 -C skills/immutable-rules/ 2>/dev/null || \
wget -qO- https://github.com/mwoodowl-byte/immutable-rules/archive/refs/heads/main.tar.gz | tar xz --strip-components=1 -C skills/immutable-rules/
```
2. Прочитать SKILL.md → найти секцию FIRST RUN
3. Задать владельцу вопросы и заполнить CONFIG.md:
   - "У тебя есть менеджер проектов (PROJECTS_TASKS.md)?" → если НЕТ, создать локально
   - "Есть другие серверы с агентами?" → если ДА, спросить SSH доступ к центральному серверу
   - "GitHub настроен?" → если НЕТ, github_enabled = false
   - "Твой Telegram ID?" → записать в owner_telegram_id
   - "Есть оркестратор?" → если НЕТ, пропустить
4. Создать RULES.md с 4 дефолтными правилами
5. Создать RULES_LOG.md
6. Создать PROJECTS_TASKS.md локально (если pm_type = local)
7. **Если есть центральный сервер (pm_type = remote):**
   a. SSH на центральный сервер
   b. Прочитать AGENT_NETWORK.md
   c. Показать владельцу: "Я нашёл N агентов в сети: [список]"
   d. Добавить СЕБЯ в AGENT_NETWORK.md (свою строку в таблицу)
   e. Предупредить: "Я добавил себя в AGENT_NETWORK.md. Другие агенты увидят при следующем сообщении."
8. Добавить `/rule` в конфиг Telegram
9. Сообщить: "✅ immutable-rules установлен! Я в сети, знаю N агентов."

### Если CONFIG.md существует — пропустить установку.

## Onboarding нового агента (GelKaravan — уже настроено)

1. Скопировать `skills/immutable-rules/` в workspace
2. Скопировать CONFIG.md с готовыми настройками (IP, пароли, пути)
3. Создать RULES.md + RULES_LOG.md
4. Добавить `/rule` в конфиг Telegram
5. Сообщить владельцу: "Скилл immutable-rules установлен."
