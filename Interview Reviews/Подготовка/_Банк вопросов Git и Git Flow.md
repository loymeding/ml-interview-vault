---
tags: [interview-prep, question-bank, git, git-flow]
type: topic-question-bank
topic: Git и Git Flow
status: active
created: 2026-07-13
cssclasses: [wide-page]
---

# 🌿 Банк вопросов — Git и Git Flow

> [!success] Развёрнутые ответы
> [[Interview Reviews/Подготовка/Git и Git Flow — развёрнутые ответы|Открыть карточку с готовыми ответами]]: она сгруппирована в восемь смысловых блоков, которые покрывают все вопросы ниже.

> [!tip] Фокус
> Здесь вопросы не про “что такое Git”, а про то, что чаще спрашивают у разработчика/ML-инженера: ветки, merge/rebase, конфликты, pull request, undo-команды и командную схему работы.

---

## 🧭 Карта блоков

| Блок | Что проверяют | Приоритет |
|---|---|---|
| Базовая модель Git | commit, index, working tree, HEAD | 🔥 высокий |
| Ветки и история | branch, merge, rebase, fast-forward | 🔥 высокий |
| Конфликты | причины, разрешение, профилактика | 🔥 высокий |
| Undo-команды | restore, reset, revert, checkout | 🔥 высокий |
| Remote workflow | fetch, pull, push, PR/MR | 🔥 высокий |
| Git Flow | feature/release/hotfix, trunk-based trade-off | 🟡 средний |
| Практика команды | code review, squash, cherry-pick, tags | 🟡 средний |

---

## 1. Базовая модель Git

- [ ] Что такое commit в Git?
- [ ] Чем отличаются working directory, staging area/index и repository?
- [ ] Что такое `HEAD`?
- [ ] Что происходит при `git add`?
- [ ] Что происходит при `git commit`?
- [ ] Чем `git status` отличается от `git diff`?
- [ ] Чем `git diff` отличается от `git diff --staged`?
- [ ] Почему Git хранит снимки состояния, а не просто набор diff-ов?
- [ ] Что такое `.gitignore` и почему уже закоммиченный файл не исчезает из Git после добавления в `.gitignore`?

---

## 2. Ветки, merge и rebase

- [ ] Что такое branch в Git?
- [ ] Чем локальная ветка отличается от remote-tracking ветки?
- [ ] Что такое fast-forward merge?
- [ ] Чем `merge` отличается от `rebase`?
- [ ] Когда лучше использовать `merge`, а когда `rebase`?
- [ ] Почему не стоит делать `rebase` публичной ветки, с которой уже работают другие люди?
- [ ] Что такое merge commit?
- [ ] Что такое squash merge и когда он удобен?
- [ ] Как перенести несколько коммитов из одной ветки в другую?
- [ ] Что такое `cherry-pick` и когда его используют?

---

## 3. Конфликты

- [ ] Почему возникают merge conflicts?
- [ ] Как Git показывает конфликт в файле?
- [ ] Как правильно разрешить конфликт?
- [ ] Что делать после ручного разрешения конфликта?
- [ ] Как отменить неудачный merge?
- [ ] Как отменить неудачный rebase?
- [ ] Почему конфликты чаще возникают в long-lived branches?
- [ ] Как уменьшить вероятность конфликтов в командной работе?

---

## 4. Undo-команды и восстановление

- [ ] Чем `git restore`, `git reset` и `git revert` отличаются друг от друга?
- [ ] Как убрать файл из staging area, но оставить изменения в рабочей директории?
- [ ] Как отменить локальные изменения в файле?
- [ ] Как безопасно отменить уже запушенный commit?
- [ ] Чем `git reset --soft`, `git reset --mixed` и `git reset --hard` отличаются?
- [ ] Почему `git reset --hard` опасен?
- [ ] Что такое `git reflog` и когда он спасает?
- [ ] Как восстановить удаленную ветку или потерянный commit через `reflog`?

---

## 5. Remote workflow и pull request

- [ ] Чем `git fetch` отличается от `git pull`?
- [ ] Что делает `git pull` под капотом?
- [ ] Чем `origin/main` отличается от `main`?
- [ ] Что происходит при `git push`?
- [ ] Почему push может быть rejected?
- [ ] Что такое pull request / merge request?
- [ ] Что обычно проверяют на code review?
- [ ] Зачем нужны CI checks перед merge?
- [ ] Что такое protected branch?
- [ ] Как синхронизировать свою feature-ветку с актуальным `main`?

---

## 6. Git Flow и альтернативные стратегии

- [ ] Что такое Git Flow?
- [ ] Какие ветки обычно есть в Git Flow: `main`, `develop`, `feature`, `release`, `hotfix`?
- [ ] Как выглядит жизненный цикл feature-ветки в Git Flow?
- [ ] Для чего нужна release-ветка?
- [ ] Для чего нужна hotfix-ветка?
- [ ] Какие плюсы у Git Flow?
- [ ] Какие минусы у Git Flow?
- [ ] Чем Git Flow отличается от trunk-based development?
- [ ] Когда trunk-based development может быть лучше Git Flow?
- [ ] Как feature flags помогают работать ближе к trunk-based подходу?

---

## 7. Теги, релизы и версионирование

- [ ] Что такое tag в Git?
- [ ] Чем lightweight tag отличается от annotated tag?
- [ ] Как теги используют для релизов модели/сервиса?
- [ ] Что такое semantic versioning на базовом уровне?
- [ ] Как связать Git tag, Docker image и версию модели в ML-проекте?

---

## 8. Практика для ML/Data Science проектов

- [ ] Что стоит хранить в Git, а что лучше выносить в DVC/S3/MLflow/artifact storage?
- [ ] Почему большие датасеты и модели не стоит коммитить в обычный Git?
- [ ] Как версионировать эксперименты, если код в Git, а данные и модели вне Git?
- [ ] Что такое Git LFS и когда он может быть полезен?
- [ ] Как оформить PR с изменением ML-пайплайна, чтобы ревьюеру было понятно влияние на метрики?
- [ ] Почему важно фиксировать seed, config и commit hash для воспроизводимости эксперимента?

---

## ✅ Следующий шаг

> [!success] Как превращать в ответы
> Лучший формат для этой темы — короткая карточка-шпаргалка: “что делает команда”, “когда использовать”, “чем опасна”. Особенно полезно отдельно расписать `merge vs rebase`, `reset vs revert`, Git Flow и типовой PR workflow.
