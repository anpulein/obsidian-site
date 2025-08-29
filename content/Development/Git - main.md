---
title: Git - main
aliases:
  - Git - main
linter-yaml-title-alias: Git - main
date created: Monday, May 12th 2025, 7:38:37 am
date modified: Wednesday, August 13th 2025, 11:02:52 am
tags:
  - type/area
  - area/development
  - impl/git
  - learn
---
## 🚀 Введение

Git — распределённая система управления версиями (DVCS), созданная Линусом Торвальдсом в 2005 году. Она позволяет отслеживать изменения в файлах и координировать работу над проектами среди нескольких разработчиков.

> **Цитата:** "Git нацелен на скорость, целостность данных и поддержку нелинейных разработческих процессов." — Linus Torvalds

## ✨ Ключевые понятия

- **Репозиторий (Repository):** Хранилище кода и истории изменений.    
- **Коммит (Commit):** Зафиксированный снимок состояния файлов.
- **Ветка (Branch):** Отдельная линия разработки.
- **HEAD:** Указатель на текущий коммит.
- **Remote:** Удалённый репозиторий.

## 🛠 Установка и настройка

```bash
# На Ubuntu/Debian\sudo apt update
sudo apt install git

# На macOS через Homebrew
brew install git

# Настройка имени и email
git config --global user.name "Ваше Имя"
git config --global user.email "you@example.com"
```

> [!TIP]  
> Проверьте настройки: `git config --list`

## 📂 Основной рабочий процесс

```bash
# Инициализация пустого репозитория
git init

# Клонирование существующего
git clone https://github.com/user/repo.git

# Проверка статуса изменений
git status

# Добавление файлов в индекс
git add <file>

# Создание коммита
git commit -m "Описание изменений"

# Отправка на сервер
git push origin main

# Получение обновлений
git pull
```

> [!TIP]  
> Используйте `git status` после каждого ключевого шага, чтобы отслеживать состояние рабочей копии.

## 🌿 Работа с ветками

1. Создать ветку:
    ```bash
    git branch feature/awesome
    ```
    
2. Переключиться на ветку:
    ```bash
    git checkout feature/awesome
    ```
    
3. Создать и сразу перейти:
    ```bash
    git checkout -b bugfix/issue-123
    ```
    
4. Слить ветку:
    ```bash
    git checkout main
    git merge feature/awesome
    ```
    
5. Удалить локальную ветку:
    ```bash
    git branch -d feature/awesome
    ```

## 🌐 Удалённые репозитории

- **Добавить remote:**
    ```bash
    git remote add origin https://github.com/user/repo.git
    ```
    
- **Список remote:**
    ```bash
    git remote -v
    ```
    
- **Переименовать:**
    ```bash
    git remote rename origin upstream
    ```
    
- **Удалить:**
    ```bash
    git remote remove upstream
    ```

## 🔧 Разрешение конфликтов

> [!WARNING]  
> При одновременном изменении одних строк в разных ветках могут возникать конфликты:
> 
> ```diff
> <<<<<<< HEAD
> ваш код
> =======
> чужой код
> >>>>>>> branch-name
> ```
> 
> После ручной правки:
> 
> ```bash
> git add [file] 
> 
> git commit
> ```

## 📋 Полезные команды

|Команда|Описание|
|---|---|
|`git log --oneline --graph`|История с ветвлением в компактном виде|
|`git diff`|Показать отличия между версиями|
|`git stash`|Временно сохранить незакоммиченные изменения|
|`git stash pop`|Восстановить сохранённые изменения|
|`git revert <commit>`|Отменить конкретный коммит новым коммитом|

## 🔗 Ссылки и доп. ресурсы

- Официальный сайт: [https://git-scm.com](https://git-scm.com/)
- Pro Git (книга): [https://git-scm.com/book/ru/v2](https://git-scm.com/book/ru/v2)
- [[Zettelkasten]] — концепция ведения заметок в Obsidian

---

_Раздел создан с использованием методик Zettelkasten: атомарность, двунаправленные связи и прогрессивная суммаризация._
