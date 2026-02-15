---
title: "MCP паритет между Claude Code и Craft Agents"
type: insight
status: active
date: 2026-02-15
author: claude-code
project: agency-wide
tags: [architecture, automation, devops]
related: [best-practice-craft-agents-windows-setup.md]
---

# MCP паритет между Claude Code и Craft Agents

## Проблема

Craft Agents имел 12 подключенных источников данных (MCP серверы, API, файловые системы), а Claude Code — 0. Это создавало разрыв в возможностях: стратегические задачи делались только в Craft Agents.

## Решение

1. Проанализировали все 12 источников Craft Agents по типу подключения
2. Определили 6 MCP-совместимых серверов (stdio + HTTP)
3. Подключили через `~/.claude.json` → `mcpServers`
4. Добавили Memory MCP для персистентной памяти
5. Скопировали 5 уникальных скиллов в единую skills-library

## Результаты

- MCP серверов у Claude Code: 0 → 7
- Общих скиллов в skills-library: 131 → 136+ (87 файлов добавлено)
- Оба агента теперь работают с одной базой знаний

## Когда применять

При добавлении нового AI-инструмента в экосистему — сразу подключать к общим источникам данных и единой skills-library.

## Когда НЕ применять

Если инструмент одноразовый и не входит в постоянный рабочий процесс.
