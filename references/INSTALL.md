# INSTALL.md — Полная инструкция по установке immutable-rules

## Быстрая установка (GelKaravan — уже всё настроено)

```bash
# 1. Скопировать скилл в workspace
cp -r immutable-rules/ ~/.openclaw/workspace/skills/

# 2. Скопировать файлы в workspace корень
cp immutable-rules/dist/RULES.md ~/.openclaw/workspace/
cp immutable-rules/dist/RULES_LOG.md ~/.openclaw/workspace/
cp immutable-rules/dist/CONFIG.md ~/.openclaw/workspace/

# 3. Добавить /rule в openclaw.json (секция channels.telegram.customCommands)
# 4. Перезапустить OpenClaw
# 5. Проверить: написать боту "покажи правила"
```

---

## Полная установка (чистый сервер, нового агента)

### Шаг 1. Установить OpenClaw

```bash
# Требуется: Node.js 18+
npm install -g openclaw
openclaw init
```

### Шаг 2. Установить скилл

```bash
# Создать директорию
mkdir -p ~/.openclaw/workspace/skills/immutable-rules/references

# Скопировать содержимое скилла (SKILL.md, references/)
# Из .skill архива или с GitHub

# Создать файлы в workspace
touch ~/.openclaw/workspace/RULES.md
touch ~/.openclaw/workspace/RULES_LOG.md
touch ~/.openclaw/workspace/CONFIG.md
```

### Шаг 3. Настроить CONFIG.md

Агент задаст вопросы автоматически при первом запуске. Или заполнить вручную:

```bash
# Минимальный CONFIG.md для агента без внешних серверов:
cat > ~/.openclaw/workspace/CONFIG.md << 'EOF'
# CONFIG.md

## Менеджер проектов
pm_type: local

## GitHub
github_enabled: false

## Владелец
owner_telegram_id: ТВОЙ_TELEGRAM_ID
owner_name: ТВОЕ_ИМЯ
EOF
```

### Шаг 4. Настроить Telegram бота

1. Создать бота через @BotFather в Telegram
2. Получить токен
3. Добавить в `openclaw.json`:

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "botToken": "ТОКЕН_ОТ_BOTFATHER",
      "allowFrom": [ТВОЙ_TELEGRAM_ID],
      "customCommands": [
        { "command": "rule", "description": "Записать/показать неизменяемое правило" }
      ]
    }
  }
}
```

### Шаг 5. Настроить GitHub (если нужен)

```bash
# 5.1. Установить Git
sudo apt-get install git -y

# 5.2. Установить GitHub CLI
# Ubuntu/Debian:
type -p wget >/dev/null || sudo apt-get install wget -y
sudo mkdir -p -m 755 /etc/apt/keyrings
wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt-get update && sudo apt-get install gh -y

# 5.3. Создать Personal Access Token
# GitHub → Settings → Developer settings → Personal access tokens → Generate new token
# Галочки: repo (весь блок)
# Ссылка: https://github.com/settings/tokens/new?scopes=repo

# 5.4. Авторизоваться
gh auth login --with-token
# Вставить токен

# 5.5. Настроить Git
git config --global user.name "твоё_имя"
git config --global user.email "твой@email.com"

# 5.6. Проверить
gh auth status

# 5.7. Обновить CONFIG.md
# github_enabled: true
# github_account: твой_github_username
```

### Шаг 6. Настроить SSH доступ (если нужен менеджер проектов на другом сервере)

```bash
# 6.1. Установить sshpass
sudo apt-get install sshpass -y

# 6.2. Проверить подключение
sshpass -p 'ПАРОЛЬ' ssh -o StrictHostKeyChecking=no root@IP "echo OK"

# 6.3. Обновить CONFIG.md
# pm_type: remote
# pm_ssh: sshpass -p 'ПАРОЛЬ' ssh -o StrictHostKeyChecking=no root@IP
# pm_path: /путь/к/PROJECTS_TASKS.md
```

### Шаг 7. Настроить workspace файлы

Минимальный набор файлов в `~/.openclaw/workspace/`:

| Файл | Обязательный | Описание |
|------|-------------|----------|
| AGENTS.md | ✅ | Правила поведения агента |
| SOUL.md | ✅ | Личность агента |
| USER.md | ✅ | Информация о владельце |
| RULES.md | ✅ | Неизменяемые правила (из скилла) |
| RULES_LOG.md | ✅ | Лог выполнения правил |
| CONFIG.md | ✅ | Конфигурация immutable-rules |
| MEMORY.md | ✅ | Долгосрочная память |
| HEARTBEAT.md | Рекомендуется | Периодические задачи |

### Шаг 8. Запустить и проверить

```bash
# Запустить OpenClaw
openclaw gateway start

# Проверить в Telegram:
# 1. Написать "покажи правила" → должен показать 4 правила
# 2. Написать "/rule" → должен войти в режим записи правила
# 3. Дать тестовую задачу → должен пройти полный workflow
```

---

## Устранение проблем

### "Агент не видит правила"
- Проверить что RULES.md в workspace корне
- Проверить что скилл в `skills/immutable-rules/SKILL.md`
- Перезапустить OpenClaw

### "Агент игнорирует правила"
- Проверить RULES_LOG.md — есть ли записи
- Проверить CONFIG.md — правильный owner_telegram_id
- Правила работают только для сообщений от владельца

### "Не подключается к менеджеру проектов"
- Проверить SSH: `sshpass -p 'ПАРОЛЬ' ssh root@IP "cat /путь/PROJECTS_TASKS.md"`
- Проверить CONFIG.md: pm_type, pm_ssh, pm_path
- Если сервер недоступен — переключить на pm_type: local

### "GitHub коммиты не работают"
- `gh auth status` — авторизован?
- `git config --global user.name` — настроено?
- CONFIG.md: github_enabled: true?
