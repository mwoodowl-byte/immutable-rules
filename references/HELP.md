# HELP.md — Справка по скиллу immutable-rules

> Этот файл читается агентом когда владелец спрашивает:
> "как добавить агента", "как настроить", "как обновить конфиг",
> "инструкция по скиллу", "help immutable-rules", "добавь агента"

---

## 🔧 АГЕНТУ: Добавить нового агента (полный алгоритм)

Когда владелец просит "добавь нового агента на сервер X" — выполнить:

### Шаг 0. Уточнить у владельца
- IP нового сервера
- SSH пароль (root)
- Имя нового агента
- Telegram бот токен (от @BotFather)

Если владелец не дал токен — попросить: "Мне нужен токен бота от @BotFather. Создай бота и пришли токен."

### Шаг 1. Проверить доступность
```bash
sshpass -p 'ПАРОЛЬ' ssh -o StrictHostKeyChecking=no root@IP "uname -a"
```
Если не подключается — сообщить владельцу и ждать.

### Шаг 2. Установить OpenClaw на новый сервер
```bash
sshpass -p 'ПАРОЛЬ' ssh -o StrictHostKeyChecking=no root@IP bash -s << 'REMOTE'
# Проверить Node.js
node -v || (curl -fsSL https://deb.nodesource.com/setup_22.x | bash - && apt-get install -y nodejs)

# Установить OpenClaw
npm install -g openclaw

# Создать workspace
mkdir -p ~/.openclaw/workspace/skills/immutable-rules/references
mkdir -p ~/.openclaw/workspace/memory

# Установить sshpass
apt-get install -y sshpass git
REMOTE
```

### Шаг 3. Скопировать immutable-rules скилл
```bash
# Скопировать скилл с текущего сервера на новый
sshpass -p 'ПАРОЛЬ_НОВОГО' scp -o StrictHostKeyChecking=no -r \
  /путь/к/skills/immutable-rules/* \
  root@IP_НОВОГО:/root/.openclaw/workspace/skills/immutable-rules/
```

### Шаг 4. Создать RULES.md (с 4 дефолтными правилами)
Скопировать текущий RULES.md на новый сервер.

### Шаг 5. Создать RULES_LOG.md
```bash
sshpass -p 'ПАРОЛЬ' ssh root@IP "echo '# RULES_LOG.md\n(No entries yet.)' > ~/.openclaw/workspace/RULES_LOG.md"
```

### Шаг 6. Создать CONFIG.md для нового агента
```bash
sshpass -p 'ПАРОЛЬ' ssh root@IP "cat > ~/.openclaw/workspace/CONFIG.md << 'EOF'
# CONFIG.md

## Менеджер проектов
pm_type: remote
pm_ssh: sshpass -p 'HouseTeka=110110-Semen' ssh -o StrictHostKeyChecking=no root@45.55.225.86
pm_path: /opt/openclaw_vps/workspace/PROJECTS_TASKS.md

## GitHub
github_enabled: true
github_account: mwoodowl-byte

## Владелец
owner_telegram_id: 309924638
owner_name: Сергей
EOF"
```

### Шаг 7. Настроить openclaw.json на новом сервере
```bash
sshpass -p 'ПАРОЛЬ' ssh root@IP "cat > ~/.openclaw/openclaw.json << 'EOF'
{
  \"agents\": {
    \"defaults\": {
      \"workspace\": \"/root/.openclaw/workspace\",
      \"llm\": { \"idleTimeoutSeconds\": 180 }
    }
  },
  \"channels\": {
    \"telegram\": {
      \"enabled\": true,
      \"botToken\": \"ТОКЕН_БОТА\",
      \"allowFrom\": [309924638],
      \"customCommands\": [
        { \"command\": \"rule\", \"description\": \"Записать/показать неизменяемое правило\" }
      ]
    }
  }
}
EOF"
```

### Шаг 8. Обновить AGENT_NETWORK.md на VPS DO
```bash
sshpass -p 'HouseTeka=110110-Semen' ssh root@45.55.225.86 \
  "sed -i '/^## Как добавить/a | N+1 | ИМЯ_АГЕНТА | ОПИСАНИЕ | IP | @БОТ | Отдельный |' \
   /opt/openclaw_vps/workspace/AGENT_NETWORK.md"
```
Или вручную через SSH — добавить строку в таблицу.

### Шаг 9. Запустить OpenClaw на новом сервере
```bash
sshpass -p 'ПАРОЛЬ' ssh root@IP "openclaw gateway start"
```

### Шаг 10. Проверить
Написать новому боту в Telegram: "покажи правила" — должен показать 4 правила.

### Шаг 11. Отчёт владельцу
```
✅ Новый агент [ИМЯ] установлен и работает:
- Сервер: IP
- Бот: @bot_name
- Скилл immutable-rules: установлен
- CONFIG.md: настроен
- AGENT_NETWORK.md: обновлён
- GitHub: настроен
- /rule: добавлен в меню
```

---

## 📋 Добавить проект в менеджер проектов

1. SSH на VPS DO
2. Открыть PROJECTS_TASKS.md
3. Добавить секцию:
```
### N. Название проекта
**Описание:** что это
**Статус:** Запланировано / В разработке / Работает
**Ответственный:** имя агента
**GitHub:** ссылка (если есть)
```
4. Создать PROJECT_CHANGELOG.md в папке проекта на сервере разработки
5. Создать GitHub репозиторий (если github_enabled)

---

## 🔄 Обновить конфигурацию

### Добавить сервер / изменить IP
1. SSH на VPS DO
2. Открыть AGENT_NETWORK.md
3. Внести изменения
4. Все агенты увидят при следующем сообщении

### Добавить оркестратора
1. Обновить AGENT_NETWORK.md — секция "Оркестратор"
2. Обновить CONFIG.md на каждом агенте — секция "Оркестратор"

---

## 📱 Команды для владельца

| Команда | Описание |
|---------|----------|
| `/rule` | Записать новое правило |
| "покажи правила" | Список всех правил |
| "покажи лог правил" | Лог выполнения |
| "как добавить агента" | Запустить алгоритм добавления |
| "как добавить проект" | Инструкция по проектам |
| "добавь агента на X.X.X.X" | Полная автоматическая установка |

---

## 📁 Файлы и их назначение

| Файл | Где | Кто меняет |
|------|-----|-----------|
| RULES.md | Workspace агента | Только владелец |
| RULES_LOG.md | Workspace агента | Агент (append-only) |
| CONFIG.md | Workspace агента | Владелец + агент при установке |
| PROJECTS_TASKS.md | VPS DO | Агенты (append) + владелец |
| PROJECT_PERMISSIONS.md | VPS DO | Только владелец |
| PROJECT_CHANGELOG.md | Папка проекта | Агенты (append-only) |
| AGENT_NETWORK.md | VPS DO | Владелец + агент по запросу |
