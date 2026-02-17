---
title: "Browser Agent — Setup для команды (Windows & macOS)"
type: playbook
status: active
date: 2026-02-17
author: craft-agents
tags: [automation, browser, mcp, ws-workspace, devops]
---

# Browser Agent — Setup для команды

## Что это

Browser Agent позволяет WS Workspace управлять **реальным Chrome** с авторизованными сессиями — не Playwright в изоляции, а живой браузер с твоими куками и аккаунтами.

**Репозиторий:** https://github.com/W-S-Agency/browser-agent

## Архитектура

```
WS Workspace (AI)
      │ stdio
      ▼
 MCP Server (Node.js)
      │ HTTP
      ▼
 Bridge Server (localhost:18793)
      │ WebSocket
      ▼
 Chrome Extension → реальный браузер с живыми сессиями
```

## Что умеет

- Навигация, клики, заполнение форм
- Скрейпинг с авторизацией (CRM, рекламные кабинеты)
- Скриншоты (полная страница или область)
- Выполнение JS на открытых вкладках
- Управление вкладками
- Несколько Chrome-профилей одновременно

## Установка — Windows

```bash
git clone https://github.com/W-S-Agency/browser-agent.git D:\Claude\sources\browser-agent
cd D:\Claude\sources\browser-agent\bridge && npm install
cd D:\Claude\sources\browser-agent\mcp-server && npm install && npm run build

# Установить как Windows Service (автостарт)
cd D:\Claude\sources\browser-agent\bridge
node install-service.js
```

## Установка — macOS

```bash
git clone https://github.com/W-S-Agency/browser-agent.git ~/browser-agent
cd ~/browser-agent/bridge && npm install
cd ~/browser-agent/mcp-server && npm install && npm run build

# Установить как LaunchAgent (автостарт)
cd ~/browser-agent/bridge
chmod +x install-launchd.sh && ./install-launchd.sh
```

## Chrome Extension

1. `chrome://extensions/` → Developer mode ON
2. Load unpacked → выбрать папку `extension/`
3. Значок показывает **Connected ✅**

## WS Workspace Config

**Windows:**
```json
{
  "command": "node",
  "args": ["D:\\Claude\\sources\\browser-agent\\mcp-server\\dist\\index.js"],
  "env": { "BRIDGE_URL": "http://localhost:18793" }
}
```

**macOS:**
```json
{
  "command": "node",
  "args": ["/Users/<username>/browser-agent/mcp-server/dist/index.js"],
  "env": { "BRIDGE_URL": "http://localhost:18793" }
}
```

## Проверка

```bash
curl http://localhost:18793/health
# → {"status":"ok",...}
```

## Troubleshooting

**Порт занят (Windows):** `netstat -ano | findstr :18792` → `taskkill /PID <pid> /F`

**Порт занят (macOS):** `lsof -ti :18792 | xargs kill -9`

**Extension не коннектится:** Проверить Bridge запущен → Reload extension в `chrome://extensions/`

## Полный гайд

`D:\Claude\sources\browser-agent\SETUP.md` (Windows)
`D:\Claude\sources\browser-agent\MACOS-SETUP.md` (macOS)
