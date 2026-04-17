# Rule Templates and Examples

## Priority Levels

| Priority | When to Use | Examples |
|----------|-------------|----------|
| 🔴 CRITICAL | Safety, security, data loss prevention | Never delete production data, never share passwords |
| 🟡 HIGH | Business rules, important workflows | Always check permissions before tasks |
| 🟢 NORMAL | Preferences, conventions | Reply in Russian, short messages |

## Rule Examples

### CRITICAL Rule Example
```
## Rule #1 🔴 CRITICAL
**Created:** 2026-04-17
**Status:** active
**Rule:** Никогда не удалять данные без подтверждения владельца. Всегда использовать trash вместо rm.
**Scope:** global
```

### HIGH Rule Example
```
## Rule #2 🟡 HIGH
**Created:** 2026-04-17
**Status:** active
**Rule:** Перед выполнением любой задачи из PROJECTS_TASKS.md проверять PROJECT_PERMISSIONS.md на наличие разрешения [x] Админ.
**Scope:** production
```

### NORMAL Rule Example
```
## Rule #3 🟢 NORMAL
**Created:** 2026-04-17
**Status:** active
**Rule:** Отвечать на русском языке. Без воды, коротко, по делу.
**Scope:** communication
```

## Available Scopes

- `global` — All actions
- `production` — Production, database, manufacturing
- `communication` — Telegram, messages, groups
- `security` — Passwords, keys, permissions
- `financial` — Money, payments, payouts
