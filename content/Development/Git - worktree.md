---
title: Git Worktree
aliases:
  - Git Worktree
linter-yaml-title-alias: Git Worktree
date created: Monday, May 12th 2025, 7:39:19 am
date modified: Wednesday, August 13th 2025, 11:02:55 am
tags:
  - type/area
  - area/development
  - impl/git
  - learn
---
# Git Worktree

## 🚀 Введение

Git Worktree — встроенная функция Git (начиная с версии 2.5), позволяющая создавать несколько рабочих копий (деревьев) одного репозитория в разных каталогах. Все они используют общий каталог `.git`, что экономит дисковое пространство и время.

> **Почему это важно?**
> 
> - Параллельная работа над разными ветками без постоянного переключения в одном каталоге.
> - Быстрый доступ к «чистой» среде для тестирования или деплоя.
> - Отсутствие дополнительных клонирований.

---

## ✨ Ключевые понятия

- **Worktree** — отдельная директория с рабочей копией ветки.
- **Главный (main) worktree** — изначальная директория репозитория.
- **Связанный worktree** — дополнительная копия, подключённая через `git worktree add`.
- **Lock-файл** — предотвращает конфликтные удаления.

---

## 🛠 Основные команды

|Команда|Описание|
|---|---|
|`git worktree add <путь> [<ветка>]`|Создать новый worktree (ветка создаётся при необходимости).|
|`git worktree list`|Показать все текущие worktree с ветками и путями.|
|`git worktree remove <путь>`|Удалить запись worktree (после ручного удаления папки).|
|`git worktree prune`|Очистить устаревшие записи удалённых worktree.|

```bash
# Пример: создаём новую ветку feature/login в отдельной папке
git worktree add ../feature/login feature/login

# Просматриваем все worktree
git worktree list

# Закрываем worktree после работы
git worktree remove ../feature/login
# (предварительно удалить саму папку)
```

---

## 🧩 Пример рабочего процесса

1. В корневом каталоге репозитория:
    ```bash
    cd ~/projects/my-repo
    ```
    
2. Создаём worktree для релизной ветки:
    ```bash
    git worktree add ../release-v2 release/v2.0
    ```
    
3. Переходим и выполняем сборку:
    ```bash
    cd ../release-v2
    ./build.sh && ./deploy.sh
    ```
    
4. Параллельно в основном каталоге продолжаем разработку в `main` или `dev`:
    ```bash
    cd ~/projects/my-repo
    git checkout dev
    git add . && git commit -m "Новая фича"
    ```
    
5. После завершения удаляем worktree:    
    ```bash
    cd ~/projects
    rm -rf release-v2
    cd my-repo
    git worktree prune
    ```

---

## 💡 Советы и лучшие практики

- Используйте чёткую схему именования папок: `feature/<имя>`, `bugfix/<номер>`, `release/vX.Y`.
- Убедитесь, что перед удалением worktree вы не потеряете незакоммиченные изменения.
- Запускайте `git worktree prune` для чистки устаревших записей.
- Не подключайте два worktree к одной и той же ветке.

---

## 🔗 Дополнительные ресурсы

- Официальная документация Git Worktree: [https://git-scm.com/docs/git-worktree](https://git-scm.com/docs/git-worktree)
- Статья «Расширенные возможности Git»: [https://git-scm.com/book/ru/v2/Git-Branching-Worktrees](https://git-scm.com/book/ru/v2/Git-Branching-Worktrees)

---

_Atomic note: Git Worktree — мощный инструмент для параллельной работы и изоляции окружений._