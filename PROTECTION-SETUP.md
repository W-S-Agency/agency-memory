# 🛡️ Защита памяти от ошибок - ИНСТРУКЦИЯ НАСТРОЙКИ

**Требуется:** Admin доступ на GitHub

---

## 1️⃣ BRANCH PROTECTION (Требуется PR для любых изменений)

### На GitHub репо:

1. Перейди: **Settings → Branches**
2. Нажми: **Add rule**
3. Заполни форму:

```
Branch name pattern: main

✓ Require a pull request before merging
  ├─ Require approvals: 1 (минимум 1 человек должен review)
  └─ Require review from Code Owners: ✓ (если CODEOWNERS существует)

✓ Require status checks to pass before merging
  └─ (Добавим GitHub Actions позже)

✓ Include administrators: ✓ (даже ты не можешь push прямо)

✓ Dismiss stale pull request approvals: ✓

✓ Require conversation resolution before merging: ✓
  (Нужно закрыть все discussions перед merge)
```

4. Нажми: **Create**

**Результат:** Никто не может напрямую пушить в main!

---

## 2️⃣ CODE OWNERS (Автоматический review)

### Создать файл CODEOWNERS:

Путь: `.github/CODEOWNERS`

```
# Все изменения должны быть reviewed тобой
* @Team588

# Playbooks - только ты можешь approve
/playbooks/ @Team588

# Best practices - только ты
/memory-exports/best-practices/ @Team588

# Insights - коллеги могут добавлять, но нужен review
/memory-exports/insights/ @Team588

# Templates - только ты
/templates/ @Team588
```

**Команды:**
```bash
cd ~/agency-memory
mkdir -p .github
cat > .github/CODEOWNERS << 'EOF'
# Default owner for everything
* @Team588

# Specific owners for critical files
/playbooks/ @Team588
/memory-exports/best-practices/ @Team588
/templates/ @Team588
EOF

git add .github/CODEOWNERS
git commit -m "Add: CODEOWNERS for branch protection"
git push origin main
```

---

## 3️⃣ PR TEMPLATE (Гайдлайны для коллег)

### Создать файл PR Template:

Путь: `.github/pull_request_template.md`

```bash
cat > .github/pull_request_template.md << 'EOF'
# 📝 Pull Request

## 📋 Описание

Что ты добавляешь или меняешь?

## 🎯 Тип изменения

- [ ] Новый инсайт (insights/)
- [ ] Обновление best-practices
- [ ] Исправление опечатки
- [ ] Другое (укажи):

## ✅ Чек-лист

- [ ] Я прочитал CONTRIBUTING.md
- [ ] Название файла понятное и описательное
- [ ] Содержимое написано на русском или английском
- [ ] Нет чувствительной информации (секреты, пароли, API ключи)
- [ ] Файл в правильной папке
- [ ] Тестировал что файл открывается и читается
- [ ] Commit message описывает что и почему

## 🔍 Review Checklist

- Информация верная и полезная?
- Формат соответствует стандарту?
- Нет противоречий с существующей памятью?
- Можно ли улучшить формулировку?

---

*Спасибо за вклад в нашу коллективную память! 🧠*
EOF

git add .github/pull_request_template.md
git commit -m "Add: PR template with guidelines"
git push origin main
```

---

## 4️⃣ CONTRIBUTING.md (Правила вклада)

### Создать файл с правилами:

```bash
cat > CONTRIBUTING.md << 'EOF'
# 🤝 Как помочь Память?

## ✅ Что можешь добавить?

### Insights (решения и уроки)
```markdown
# [Название проблемы/решения]

## Проблема
Что было неправильно?

## Решение
Как это решить?

## Результаты
Какой был результат? (+X%, -Y минут, и т.д.)

## Дата
2026-02-05

## Автор
@твой-github-username
```

### Best Practices (улучшение существующих)
- Только обновляй, если ты эксперт в этом
- Приводи примеры и данные
- Получи review перед merge

### Templates (новые шаблоны)
- Только утвержденные шаблоны
- Должны быть протестированы
- С примерами заполнения

## ❌ Что НЕ добавлять?

```
❌ Секреты (пароли, API ключи, токены)
❌ Персональные данные коллег
❌ Чувствительную информацию клиентов
❌ Большие бинарные файлы (>10MB)
❌ Копипасту без атрибуции
❌ Некорректную информацию
❌ Spam или рекламу
```

## 📝 Процесс

1. Создаешь новый branch: `git checkout -b add-my-insight`
2. Добавляешь файл в правильную папку
3. Пишешь хороший commit message
4. Пушишь: `git push origin add-my-insight`
5. На GitHub: создаешь Pull Request
6. Ждешь review (обычно 24 часа)
7. После approval → merge!

## 💡 Примеры хороших инсайтов

### Плохо ❌
```
# Новый способ

Это лучше чем раньше.
```

### Хорошо ✅
```
# Оптимизация кэширования в Redis

## Проблема
При большом объеме данных Redis использовал 100GB памяти.

## Решение
Установили TTL 24 часа и compression.

## Результаты
- Память: 100GB → 30GB (-70%)
- Скорость: +15% благодаря меньшему размеру
- Дата: 2026-02-05
- Проект: ClientName/ProjectX
```

## ❓ Вопросы?

Создай Issue или спроси в Slack #agency-memory
EOF

git add CONTRIBUTING.md
git commit -m "Add: Contribution guidelines"
git push origin main
```

---

## 5️⃣ GITHUB ACTIONS (Автоматические проверки)

### Создать workflow для проверок:

```bash
mkdir -p .github/workflows
cat > .github/workflows/pr-checks.yml << 'EOF'
name: PR Checks

on: [pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Check for secrets
        run: |
          if grep -r "password\|secret\|token\|api[_-]key" --include="*.md" . 2>/dev/null; then
            echo "❌ Найдены потенциальные secrets!"
            exit 1
          fi

      - name: Check file sizes
        run: |
          find . -type f -size +50M -print | while read file; do
            echo "❌ Файл слишком большой: $file"
            exit 1
          done

      - name: Validate markdown
        run: |
          find . -name "*.md" -type f | while read file; do
            if ! head -c 3 "$file" | grep -q ''; then
              echo "❌ Невалидный markdown: $file"
              exit 1
            fi
          done

      - name: Comment on PR
        if: success()
        run: echo "✅ Все проверки пройдены!"
EOF

git add .github/workflows/pr-checks.yml
git commit -m "Add: GitHub Actions PR validation"
git push origin main
```

---

## 6️⃣ README.md для коллег

Обновить README с инструкциями:

```bash
cat >> README.md << 'EOF'

## 🛡️ Как не испортить память?

### ✅ Делай
- ✓ Добавляй инсайты в `memory-exports/insights/`
- ✓ Пиши на русском или английском
- ✓ Приводи примеры и данные
- ✓ Получи review перед merge

### ❌ НЕ делай
- ❌ Не пушь секреты (пароли, ключи, токены)
- ❌ Не удаляй playbooks или best-practices
- ❌ Не меняй структуру без обсуждения
- ❌ Не добавляй большие файлы

### 📖 Прочитай
- [CONTRIBUTING.md](CONTRIBUTING.md) - Правила вклада
- [.github/CODEOWNERS](.github/CODEOWNERS) - Кто reviewing
- [.github/pull_request_template.md](.github/pull_request_template.md) - PR шаблон

EOF

git add README.md
git commit -m "Update: Add protection guidelines to README"
git push origin main
```

---

## 🔒 ИТОГОВАЯ ЗАЩИТА

```
✅ Branch Protection        → Все через PR
✅ Code Owners             → Автоматический review
✅ PR Template             → Гайдлайны
✅ Contributing.md         → Правила
✅ GitHub Actions          → Автоматизм
✅ README                  → Инструкции
✅ Git History             → Отката возможна

РЕЗУЛЬТАТ: 🛡️ Память защищена от ошибок!
```

---

## 📋 КОМАНДЫ ДЛЯ БЫСТРОЙ НАСТРОЙКИ

```bash
cd ~/agency-memory

# 1. Создать все защиты
mkdir -p .github/workflows

# 2. CODEOWNERS
cat > .github/CODEOWNERS << 'EOF'
* @Team588
/playbooks/ @Team588
/memory-exports/best-practices/ @Team588
/templates/ @Team588
EOF

# 3. PR Template
cat > .github/pull_request_template.md << 'EOF'
# Pull Request
...
EOF

# 4. Contributing
cat > CONTRIBUTING.md << 'EOF'
# Contributing
...
EOF

# 5. Коммит и push
git add .github/ CONTRIBUTING.md
git commit -m "Add: Multi-level protection against errors"
git push origin main

# 6. На GitHub:
# Settings → Branches → Add rule → main branch
# Требова PR, require approvals, require CODEOWNERS review
```

---

## ✨ Результат

**Коллега пытается добавить инсайт:**

```
1. git checkout -b add-my-insight
2. Добавляет файл
3. git push origin add-my-insight
4. На GitHub: Create PR

GitHub автоматически:
✓ Запрашивает review у @Team588
✓ Запускает проверки (secrets, format, etc)
✓ Требует minimum 1 approval перед merge

Ты:
1. Проверяешь информацию
2. Запрашиваешь исправления если нужны
3. Approve → Merge

🎉 Инсайт добавлен безопасно!
```

---

*Начни с ШАГ 1 (Branch Protection) на GitHub*
