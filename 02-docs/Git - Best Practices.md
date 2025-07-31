---
author: Олег Дурыгин
tags: 
date: 2025-07-21
---

## Рекомендации
Обязательно создайте карточку в Github Projects (с помощью своего TeamLead) перед тем как приступать к задаче по написанию кода

## Название веток
Называть ветку следует в соответствии с [Convention Commit](https://www.conventionalcommits.org/ru/v1.0.0/) (для [PyCharm](https://plugins.jetbrains.com/plugin/13389-conventional-commit), [VScode](https://marketplace.visualstudio.com/items?itemName=vivaxy.vscode-conventional-commits) есть специальный плагин ), а также использовать номер задачи в Github Projects и ее краткое название
1. feat/cv-#/short-name-of-task
2. fix/cv-#/short-name-of-task

Предлагаемые теги:
- cv-pipe
- cv-rnd (можно объединить в один cv)
- infra
- ds

## Коммиты

### Общее правило
Коммиты следует писать **на английском языке**, используя **Conventional Commit** формат.

### Conventional Commit

#### Структура сообщения коммита
```
<type>: <short description>  

[optional body]

[optional footer]
```
- `<type>` - тип изменения
- `<short description>` - краткое описание изменений (с маленькой буквы, без точки в конце)
- `body` - дополнительное описание (многострочное), начиная со второй строки
- `footer` - служебная информация (например, `BREAKING CHANGE`, `Closes #123`)

#### Описание структуры
- `<type>`
	**Основные типы:**
	    - **fix**: исправление ошибки (bug) в коде — соответствует `PATCH` в SemVer.
		- **feat**: добавление новой функциональности (feature) — соответствует `MINOR` в SemVer.
		- **docs**: изменения в документации.
		- **style**: изменения, не влияющие на поведение кода (пробелы, форматирование и т. д.).
		- **refactor**: изменение кода без исправлений багов или добавления функциональности.
		- **perf**: улучшения производительности.
		- **test**: добавление или изменение тестов.
		- **build**: изменения, связанные со сборкой проекта.
		- **ci**: изменения в CI/CD.
		- **chore**: рутинные задачи (например, изменение `.gitignore`).
		- **revert**: откат предыдущего коммита.
- `<short description>` 
	**Краткое описание:**
	- Формулируется с маленькой буквы
	- Без точки в конце
	- Является заголовком коммита
- `<body>`
	**Подробное описание** (необязательное):
	- Многострочный текст, начинается со второй строки
	- Используется для объяснения причины изменений, технических деталей, ограничений и т.п.
	- Может включать код, списки, Markdown
-  `<footer>` 
	**Служебный блок** (необязательный):
	- **BREAKING CHANGE**: коммит, содержащий текст `BREAKING CHANGE:` в начале тела (body - начиная со 2-й строки) сообщения или в подвале (footer - в самом низу). Такие изменения нарушают обратную совместимость API — соответствуют `MAJOR` в SemVer. .
	- Ссылок на тикеты (Closes #123, Refs #456)

#### Примеры
- `feat: add new functionality`
- `fix: fix token validation issue`
- `docs: update README with usage examples`
- `refactor: simplify validation logic`

## Pull Request / Merge Request

### Название PR/MR
Название должно содержать **номер задачи** из GitHub Projects, **тип коммита по Conventional Commit**, и **краткое описание** изменений. Пример:
- `[CV-#]: feat - add new functionality`

Если работа над PR/MR ещё не завершена, но вы уже открыли его, добавьте префикс **[WIP]** (Work in Progress). Пример:
- `[WIP] [CV-#]: feat - add new functionality`

### Описание PR/MR
Описание должно быть написано **на английском языке** и содержать **список логически завершённых изменений**, представленных с помощью маркированного списка (`-`). Это необходимо для корректной генерации **Changelog** в будущем.

**Пример:**
```
- Fixed an overfitting issue in the recommendation model by adding Dropout regularization.  
- Improved configuration flexibility: the Dropout rate is now a configurable parameter.  
- Expanded test coverage — added checks to ensure regularization layers are present.  
- Updated documentation: explained the Dropout mechanism and provided guidance on choosing an appropriate rate.
```

### Назначение на Code Review
- После создания PR/MR всегда **назначайте себя как Assignee** и **ревьюера как Reviewer**.
- Если вы хотите привлечь **больше одного ревьюера**, добавьте комментарий к PR/MR с упоминанием нужных участников (`@username`), так как в бесплатной версии GitHub можно выбрать только одного Reviewer.
- Если вы не знаете, кого назначить — укажите **Team Lead'а** в ревьюеры.

### Комментарии к PR / MR
Это единственное место где допускает РУССКИЙ язык для коммуникации.
**Правила коммуникации:**
- **Никогда не игнорируйте комментарии ревьюера.**
- Если вы **согласны** — внесите правки и поставьте 👍.
- Если **не согласны** — дайте аргументированный ответ в треде.
- **Не закрывайте** комментарий вручную — это делает только Reviewer.
- После внесения всех изменений **нажмите "Re-request review"**, чтобы уведомить ревьюера, что правки готовы.