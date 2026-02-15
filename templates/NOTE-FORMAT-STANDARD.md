# Agency Memory — Единый формат заметок v1.0

Стандарт для всех агентов: Claude Code, Craft Agents, человек.

---

## Правило #1: YAML Frontmatter обязателен

Каждый файл начинается с YAML блока. Это позволяет обоим агентам парсить метаданные автоматически.

```yaml
---
title: "Название заметки"
type: insight | decision | best-practice | playbook | template | export
status: draft | active | archived
date: 2026-02-15
updated: 2026-02-15
author: claude-code | craft-agents | human
project: project-name | agency-wide
tags: [tag1, tag2, tag3]
---
```

### Обязательные поля

| Поле | Формат | Пример |
|------|--------|--------|
| `title` | Строка в кавычках | `"Cache Invalidation Strategy"` |
| `type` | Enum | `insight`, `decision`, `best-practice`, `playbook`, `template`, `export` |
| `status` | Enum | `draft`, `active`, `archived` |
| `date` | ISO 8601 | `2026-02-15` |
| `author` | Enum | `claude-code`, `craft-agents`, `human` |

### Опциональные поля

| Поле | Формат | Когда нужен |
|------|--------|-------------|
| `updated` | ISO 8601 | При обновлении |
| `project` | slug | Если привязан к проекту |
| `tags` | Массив строк | Всегда рекомендуется |
| `supersedes` | Имя файла | Если заменяет старую заметку |
| `related` | Массив имён файлов | Если связан с другими заметками |
| `skills` | Массив skill ID | Если использовались скиллы |

---

## Правило #2: Структура по типу заметки

### Insight (Решение/Урок)

```markdown
---
title: "Название"
type: insight
status: active
date: 2026-02-15
author: claude-code
project: project-name
tags: [performance, cache, backend]
---

# Название

## Проблема

Что было неправильно? 2-3 предложения.

## Решение

1. Шаг 1
2. Шаг 2
3. Шаг 3

## Результаты

- Метрика: было X → стало Y (+Z%)
- Экономия: время/деньги/ресурсы

## Когда применять

Контекст, в котором это работает.

## Когда НЕ применять

Ограничения и исключения.
```

### Decision (Архитектурное решение)

```markdown
---
title: "Название решения"
type: decision
status: active
date: 2026-02-15
author: human
project: project-name
tags: [architecture, frontend]
---

# Название решения

## Контекст

Почему нужно было принять решение?

## Варианты

| Вариант | Плюсы | Минусы |
|---------|-------|--------|
| A | ... | ... |
| B | ... | ... |

## Решение

Выбран вариант X, потому что...

## Последствия

Что это означает для архитектуры и процессов.
```

### Best Practice (Лучшая практика)

```markdown
---
title: "Название практики"
type: best-practice
status: active
date: 2026-02-15
author: craft-agents
tags: [devops, ci-cd]
---

# Название практики

## Правило

Чёткая формулировка в 1-2 предложения.

## Почему

Обоснование с данными.

## Как применять

Конкретные шаги или примеры кода.

## Исключения

Когда правило не работает.
```

### Playbook (Процесс)

```markdown
---
title: "Название процесса"
type: playbook
status: active
date: 2026-02-15
author: human
tags: [delivery, process]
skills: [ln-200, ln-401]
---

# Название процесса

## Обзор

Цель и область применения.

## Этапы

### Этап 1: Название

**Скиллы:** `ln-200-scope-decomposer`

**Действия:**
1. ...
2. ...

**Результат:** Что получаем на выходе.

### Этап 2: Название

...

## KPI / Критерии успеха

- [ ] Критерий 1
- [ ] Критерий 2
```

---

## Правило #3: Именование файлов

```
{тип}-{описание}.md

Примеры:
  insight-cache-invalidation-strategy.md
  decision-nextjs-over-nuxt.md
  best-practice-git-commit-messages.md
  playbook-client-onboarding.md
  template-project-brief.md
```

**Правила:**
- Только латиница, lowercase
- Дефисы вместо пробелов
- Без дат в имени файла (дата в frontmatter)
- Тип заметки — префикс

---

## Правило #4: Размещение по папкам

```
agency-memory/
├── memory-exports/
│   ├── best-practices/     ← type: best-practice
│   ├── insights/           ← type: insight
│   └── decisions/          ← type: decision
├── playbooks/              ← type: playbook
└── templates/              ← type: template
```

---

## Правило #5: Автор и атрибуция

| Автор | Когда использовать |
|-------|-------------------|
| `claude-code` | Заметка создана через Claude Code (VS Code / CLI) |
| `craft-agents` | Заметка создана через Craft Agents (Desktop) |
| `human` | Заметка написана человеком вручную |

При совместной работе указывать того, кто инициировал создание.

---

## Правило #6: Теги

Используй существующие теги, не создавай дубликаты.

**Домены:** `frontend`, `backend`, `devops`, `design`, `marketing`, `seo`, `crm`, `analytics`

**Типы работ:** `architecture`, `performance`, `security`, `testing`, `automation`, `process`

**Проекты:** `2penguins`, `wk-connect`, `ws-agency`, `topholz24`

**Инструменты:** `nextjs`, `react`, `sanity`, `vercel`, `bitrix24`, `figma`, `playwright`

---

## Правило #7: Обновление существующих заметок

1. Обнови поле `updated` в frontmatter
2. Если заметка полностью заменяет другую — добавь `supersedes: old-file-name.md`
3. Старую заметку переведи в `status: archived`
4. Не удаляй старые файлы — архивируй

---

## Миграция старых файлов

Существующие файлы без frontmatter продолжают работать. При следующем редактировании — добавь frontmatter по этому стандарту.
