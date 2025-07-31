---
author: Василий Челпанов
tags: 
date: 2025-07-21
---
## Инструкция по работе с `pre-commit`
[pre-commit](https://pre-commit.com/#intro) - A framework for managing and maintaining multi-language pre-commit hooks.
### Установка
1. Установка
	Установи `pre-commit`, если ещё не установлен:
	```bash
	pip install pre-commit
	```

2. Создание конфигурации
	Создай файл `.pre-commit-config.yaml` в корне проекта:
	```yaml
	# Пример абстрактного хука (замени <> на реальные значения)
	- repo: <URL до репозитория с хуками>
	  rev: <тег или коммит>
	  hooks:
	    - id: <id хука>
	      name: <читаемое имя хука>
	      description: <что делает>
	      entry: <точка входа>
	      language: <язык>
	      types: [<файлы>]
	      stages: [<pre-commit|commit-msg|prepare-commit-msg>]
	```
	
	Пример:
	```yaml
	repos:
	  - repo: https://github.com/pre-commit/pre-commit-hooks
	    rev: v4.5.0
	    hooks:
	      - id: trailing-whitespace
	        description: Remove trailing whitespace in files
	      - id: end-of-file-fixer
	        description: Ensure files end with a single newline
	```
	
3. Установка хуков
	```bash
	# ↓ pre-commit hooks
	pre-commit install
	# ↓ commit-msg hooks
	pre-commit install --hook-type commit-msg 
	# ↓ pre-commit, commit-msg, prepare-commit-msg, etc. hooks
	pre-commit install --hook-type all
	```

   > [!note]
    > Если был добавлен новый хук (или изменился `hook-type`), **обязательно переустанови хуки**.

5. Проверка вручную
	```bash
	pre-commit run --all-files
	```

 6. Обновление версий хуков
	```bash
	pre-commit autoupdate
	```

### Заметки
- `pre-commit` запускает хуки автоматически при коммите.
- Работает изолированно, кэширует окружение для каждого хука.
- Поддерживает локальные и удалённые хуки.

## Git Hook: Conventional Commits

### Описание
Этот pre-commit хук проверяет, что сообщение коммита соответствует [Conventional Commits](https://www.conventionalcommits.org/ru/v1.0.0/).

### Подключение из удалённого репозитория
```
repos:
  - repo: <repo-path>
    rev: v1.0.0
    hooks:
      - id: commit-msg-conventional
```

### Примеры
Корректные сообщения
```bash
feat: add user authentication
fix(api): handle empty token
docs(readme): update launch instructions
```
Некорректные сообщения:
```bash
fix:add login                # нет пробела после :
feat: Add login              # описание с заглавной буквы
hotfix: critical fix         # недопустимый тип
```
