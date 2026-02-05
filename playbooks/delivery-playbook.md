# 🚀 Delivery Playbook

Пошаговый процесс для разработки и доставки проектов.

## День 1: Планирование

**Используй skills:**
- `ln-200-scope-decomposer` - Разбить на Epics и Stories
- `ln-300-task-coordinator` - Создать Tasks с оценками

**Действия:**
1. Проанализируй requirements
2. Создай Epics (3-7 штук)
3. Создай Stories для каждого Epic
4. Создай Tasks для каждого Story
5. Оцени время и ресурсы

**Результат:** Project plan с breakdown на Stories и Tasks

---

## Дни 2-10: Разработка

**Используй skills:**
- `ln-401-task-executor` - Реализация
- `ln-402-task-reviewer` - Code review
- `ln-403-task-rework` - Доработка

**Процесс для каждого Task:**
```
1. Начало (Task In Progress)
2. Используй ln-401-task-executor (пишем код)
3. Push в PR
4. Используй ln-402-task-reviewer (review)
5. Если ошибки → ln-403-task-rework (fixes)
6. Merge → Task Done
7. Repeat для следующего Task
```

**Результат:** Готовый код, готовые к merge PR

---

## День 11: Тестирование

**Используй skills:**
- `ln-510-test-planner` - План тестов
- `ln-513-auto-test-planner` - Автотесты
- `ln-782-test-runner` - Запуск тестов

**Действия:**
1. План тестирования
2. Создай автотесты
3. Запусти все тесты
4. Исправь failed тесты
5. Добей 80%+ coverage

**Результат:** All tests passing, coverage 80%+

---

## День 12: Аудит

**Используй skills:**
- `ln-620-codebase-auditor` - Full audit
- `ln-621-security-auditor` - Security scan
- `ln-625-dependencies-auditor` - Deps check

**Действия:**
1. Full codebase audit
2. Security audit
3. Dependencies audit
4. Исправь issues
5. Approve для production

**Результат:** All audits passed

---

## День 13: Deploy

**Используй skills:**
- `deploy-vercel` или `ln-783-container-launcher`

**Действия:**
1. Настрой environment variables
2. Deploy на production
3. Проверь health checks
4. Мониторь метрики
5. Все ОК? ✅

**Результат:** Live in production, все работает

---

## 🎯 Quality Gates

```
✅ Code Review: PASS
✅ Tests: 80%+ coverage, all passing
✅ Security Audit: No critical issues
✅ Dependencies: All up-to-date
✅ Performance: < 3s load time
✅ Documentation: Complete
```

---

## 📋 Чеклист проекта

- [ ] День 1: Planning (Epics, Stories, Tasks)
- [ ] Дни 2-10: Development (All PRs merged)
- [ ] День 11: Testing (80%+ coverage)
- [ ] День 12: Audit (All passed)
- [ ] День 13: Deploy (Production)
- [ ] Документация обновлена
- [ ] Решения сохранены в памяти

---

**Версия:** 1.0
**Дата:** 2026-02-05
