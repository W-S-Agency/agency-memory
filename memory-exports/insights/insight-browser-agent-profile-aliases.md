---
title: "Browser Agent v1.1.0 - Profile Aliases для MCP"
type: insight
status: active
date: 2026-02-16
author: craft-agents
project: ws-agency
tags: [browser-agent, mcp, automation, profile-aliases, ws-workspace]
---

# Browser Agent v1.1.0 - Profile Aliases для MCP

## Проблема

При работе с Browser Agent через MCP приходилось использовать длинные неудобные ID профилей:
```javascript
browser_navigate({
  profileId: "chrome-1771258341643-hp1qlw",
  url: "https://semrush.com"
})
```

Проблемы:
- **Нечитаемость кода** - невозможно понять какой профиль используется
- **Сложность запоминания** - длинные ID нельзя запомнить
- **Ошибки в коде** - легко перепутать профили при копировании ID
- **Нет контекста** - код не самодокументируется

## Решение

Реализована система **Profile Aliases** - человекочитаемые имена для браузерных профилей:

### 1. Popup UI для установки алиасов
- Поле ввода в расширении Chrome
- Валидация формата (lowercase, numbers, hyphens)
- Сохранение в `chrome.storage.local`
- Визуальный фидбек (success/error)

### 2. HTTP API для программной установки
```bash
POST http://localhost:18793/profiles/:profileId/alias
Content-Type: application/json

{"alias": "work"}
```

### 3. Автоматическое резолвение в Bridge Server
- Mapping `alias → profileId` в памяти
- Методы `resolveProfileId()` и `updateAlias()`
- Поддержка в `getProfile()` и `executeCommand()`

### 4. Обновлённый MCP Server
- TypeScript interface `Profile` с полем `alias`
- Прозрачное резолвение для всех 12 MCP tools
- Обратная совместимость с ID

## Результаты

**До:**
```javascript
browser_execute_js({
  profileId: "chrome-1771258341643-hp1qlw",
  code: "document.title"
})
```

**После:**
```javascript
browser_execute_js({
  profileId: "work",
  code: "document.title"
})
```

### Метрики релиза
- **Версия**: v1.1.0
- **Дата**: 16 февраля 2026
- **Изменений**: +564 строк / -29 строк / 8 файлов
- **Время разработки**: ~2 часа (включая тестирование + документацию)

### Изменённые файлы
| Файл | Строк | Назначение |
|------|-------|------------|
| `extension/popup.html` | +81 | Alias UI |
| `extension/popup.js` | +72 | Alias logic |
| `extension/background.js` | +52 | Storage & sync |
| `bridge/profile-manager.js` | +78 | Resolution methods |
| `bridge/server.js` | +80 | HTTP endpoints |
| `mcp-server/src/bridge-client.ts` | +1 | TypeScript interface |
| `README.md` | updated | Feature highlights |
| `ALIASES.md` | new | Complete guide |

## Когда применять

✅ **Используй алиасы когда:**
- Работаешь с несколькими профилями (work, personal, team)
- Пишешь код для повторного использования
- Нужна читаемость и самодокументирование
- Профили могут меняться (гибкость)

**Примеры использования:**
```javascript
// Навигация
browser_navigate({ profileId: "work", url: "https://vercel.com" })

// Выполнение JS
browser_execute_js({ profileId: "personal", code: "document.title" })

// Скриншот
browser_screenshot({ profileId: "team", fullPage: true })

// Естественный язык в WS Workspace
"Открой Semrush на work профиле"
"Сделай скриншот Google Ads на personal"
```

## Когда НЕ применять

❌ **Не используй алиасы когда:**
- Разовая автоматизация (можно использовать ID напрямую)
- Профиль заведомо известен и не меняется
- Работаешь с единственным профилем

## Архитектура

```
┌─────────────────┐
│ WS Workspace    │
│ (Claude Code)   │
└────────┬────────┘
         │ MCP stdio
         ▼
┌─────────────────┐
│ MCP Server      │
│ (TypeScript)    │
└────────┬────────┘
         │ HTTP :18793
         ▼
┌─────────────────┐     WebSocket :18792      ┌──────────────────┐
│ Bridge Server   │◄──────────────────────────►│ Chrome Extension │
│ (Node.js)       │                            │ (alexanderwirt)  │
│                 │                            └──────────────────┘
│ ProfileManager: │
│ • aliases Map   │     WebSocket :18792      ┌──────────────────┐
│ • resolveId()   │◄──────────────────────────►│ Chrome Extension │
│ • updateAlias() │                            │ (team)           │
└─────────────────┘                            └──────────────────┘
```

### Компоненты

1. **Extension (Chrome)**
   - Popup UI с полем ввода
   - Валидация: `/^[a-z0-9-]+$/`, max 50 символов
   - Storage: `chrome.storage.local` (persistent)
   - Отправка при регистрации + `update_alias` message

2. **Bridge Server**
   - `Map<alias, profileId>` в памяти (runtime)
   - HTTP endpoint: `POST /profiles/:id/alias`
   - Обработка `update_alias` WebSocket message
   - Резолвение в `executeCommand()`

3. **Profile Manager**
   - `resolveProfileId(aliasOrId)` - попытка resolve
   - `updateAlias(profileId, newAlias)` - обновление mapping
   - `getProfile(aliasOrId)` - получение профиля

4. **MCP Server**
   - TypeScript interface `Profile { alias: string | null }`
   - Прозрачное резолвение для всех tools
   - Обратная совместимость

## Технические детали

### Хранение
- **Extension**: `chrome.storage.local` (переживает перезагрузку)
- **Bridge**: `Map<alias, profileId>` в памяти (очищается при рестарте)
- **Синхронизация**: автоматическая при reconnect extension

### Резолвение (приоритет)
1. Проверка что это валидный `profileId` → использовать напрямую
2. Если не найден, попытка resolve как alias → использовать `resolvedId`
3. Если ничего не найдено → `Error: Profile not found`

### Валидация
```javascript
// Regex
/^[a-z0-9-]+$/

// Примеры валидных алиасов
✅ work
✅ personal
✅ comet-ads
✅ team-profile-1

// Невалидные
❌ Work (uppercase)
❌ my_profile (underscore)
❌ профиль (кириллица)
```

## Интеграция с WS Workspace

Browser Agent уже настроен как **source** в WS Workspace:

```json
// ~/.ws-workspace/workspaces/my-workspace/sources/browser-agent/config.json
{
  "type": "mcp",
  "transport": "stdio",
  "command": "node D:\\Claude\\sources\\browser-agent\\mcp-server\\dist\\index.js",
  "env": {
    "BRIDGE_URL": "http://localhost:18793"
  },
  "connectionStatus": "connected"
}
```

**Использование:**
```javascript
// Прямые MCP вызовы
browser_list_profiles()
browser_execute_js({ profileId: "work", code: "document.title" })

// Естественный язык
"Открой Vercel на team"
"Выполни на personal: покажи заголовок"
```

## Документация

### Созданные файлы
- **ALIASES.md** - полный гайд по использованию (4.9KB)
- **README.md** - обновлён с примерами алиасов
- **GitHub Release v1.1.0** - подробный changelog

### Установка алиаса

**Метод 1: Popup UI** (рекомендуется)
1. Кликнуть на иконку Browser Agent
2. Ввести алиас (например, `work`)
3. Нажать Save
4. Увидеть ✓ "Alias 'work' saved successfully"

**Метод 2: HTTP API** (программно)
```bash
curl -X POST http://localhost:18793/profiles/chrome-xxx/alias \
  -H "Content-Type: application/json" \
  -d '{"alias": "work"}'
```

## Тестирование

### Протестированные сценарии
✅ Установка алиаса через popup UI
✅ HTTP API для программной установки
✅ Резолвение в `browser_list_profiles()`
✅ Использование в `browser_execute_js()`
✅ Использование в `browser_navigate()`
✅ Переключение между профилями по алиасам
✅ Одновременная работа с 2 профилями

### Реальные профили
- `alexanderwirt` → chrome-1771246123525-ezd05p (Google Ads, Gmail, Sheets)
- `team` → chrome-1771258341643-hp1qlw (Vercel, GitHub, Calendar)

## Уроки и insights

### 1. Google Ads блокирует автоматизацию
**Проблема**: На странице выбора аккаунта JavaScript execution заблокирован (защита от ботов)

**Решение**:
- Ручной выбор аккаунта пользователем
- Автоматизация ПОСЛЕ входа в dashboard
- Работает отлично для извлечения данных кампаний

### 2. JavaScript execution в Chrome Extensions
**Best practice**:
```javascript
await chrome.scripting.executeScript({
  target: { tabId },
  world: 'MAIN',  // Важно!
  func: (codeString) => {
    return eval(codeString);  // Простой eval работает лучше
  },
  args: [code]
});
```

**Проблемы**:
- `world: 'ISOLATED'` (по умолчанию) - нет доступа к window
- Function constructor - возвращает null
- AsyncFunction - крашит Service Worker

### 3. Extension reload после обновления кода
**Важно**: Chrome Extension не делает hot-reload автоматически

**Решение**:
1. `chrome://extensions` → Browser Agent
2. Нажать кнопку **↻ Reload**
3. Переоткрыть popup

Иначе новый функционал не будет работать.

### 4. Winston logging для production
**Отлично работает**:
- Daily rotation (14 дней general, 30 дней errors)
- Structured JSON logs
- Удобно для анализа и дебага

```javascript
logger.info('[Bridge] Profile alias updated', {
  profileId,
  alias
});
```

## Преимущества

### 1. Читаемость кода
```javascript
// До
browser_navigate({
  profileId: "chrome-1771258341643-hp1qlw",
  url: "https://semrush.com"
})

// После
browser_navigate({
  profileId: "work",
  url: "https://semrush.com"
})
```

### 2. Самодокументирование
Код понятен без комментариев - сразу видно с каким профилем работаем.

### 3. Гибкость
Можно поменять профиль не трогая код - просто переназначить алиас.

### 4. Обратная совместимость
Старый код с ID продолжает работать - никаких breaking changes.

## Все фичи Browser Agent v1.1.0

- ✅ WebSocket Authentication - Token-based security
- ✅ File Logging with Rotation - 14-day retention
- ✅ Auto-Reconnect - Exponential backoff (1s → 30s)
- ✅ Auto-Start - Windows Service / macOS LaunchAgent
- ✅ Multiple Profiles - Chrome, Comet, Arc, Brave
- ✅ **Profile Aliases** - NEW! Human-readable names
- ✅ Cross-Platform - Windows & macOS support
- ✅ Production Stability - Crash recovery, health checks

## Ссылки

- **GitHub Repo**: https://github.com/W-S-Agency/browser-agent
- **Release v1.1.0**: https://github.com/W-S-Agency/browser-agent/releases/tag/v1.1.0
- **Local Path**: `D:\Claude\sources\browser-agent`
- **Documentation**:
  - [ALIASES.md](https://github.com/W-S-Agency/browser-agent/blob/master/ALIASES.md) - Complete guide
  - [README.md](https://github.com/W-S-Agency/browser-agent/blob/master/README.md) - Overview
  - [PRODUCTION-README.md](https://github.com/W-S-Agency/browser-agent/blob/master/PRODUCTION-README.md) - Windows setup
  - [MACOS-SETUP.md](https://github.com/W-S-Agency/browser-agent/blob/master/MACOS-SETUP.md) - macOS setup

## Для команды: Upgrade на v1.1.0

```bash
# 1. Обновить код
cd D:\Claude\sources\browser-agent  # или ваш путь
git pull

# 2. Перезагрузить расширение
# chrome://extensions → Browser Agent → кнопка Reload

# 3. Перезапустить Bridge Server
# Windows:
Restart-Service BrowserAgentBridge

# macOS:
launchctl kickstart -k gui/$UID/com.browseragent.bridge

# 4. Установить алиасы через popup UI
# Кликнуть на иконку → ввести алиас → Save
```

## Status

✅ **Production-ready** - готово к использованию
✅ **Протестировано** - на 2 профилях с реальными задачами
✅ **Документировано** - полные гайды созданы
✅ **GitHub Release** - v1.1.0 опубликован

---

**Co-Authored-By**: WS Workspace <noreply@wsagency.dev>
**Created**: 2026-02-16
**Author**: Alexander Wirt via craft-agents
