## Инструкция
### 0. Установка LFS
```bash
# Установить Git LFS (если ещё не установлен)
sudo apt install git-lfs          # Debian/Ubuntu
brew install git-lfs              # macOS

```

### 1. Инициализация Git LFS

```bash
git lfs install
```

### 2. Получение GitHub Personal Access Token (PAT)

1. Перейди в GitHub: [https://github.com/settings/tokens](https://github.com/settings/tokens) 
2. Нажми кнопку `Generate new token` 
3. Введи описание 
4. Выстави срок действия токена 
5. Настрой списка репозиториев: 
	- `Resource owner` 
	  Доступ к личным репозиториям, либо доступ к репозиториям конкретной организации 
	- `Repository access` 
	  Доступ ко всем публичным репозиториям / ко всем репозиториям / к выбранным репозитория 
6. Настройте права: 
	- Необходимо выдать права `Contents` для `Read and write` доступ 
7. Нажми `Generate token` 
8. Cкопируй токен сразу — он отображается только один раз

### 3. Настройка `.lfsconfig`

В корне Git-репозитория создай файл `.lfsconfig`:

```ini
[lfs]
url = https://<giftless-domain>/<user_or_org>/<repo>.git/info/lfs
```

#### Пояснение:

|Плейсхолдер|Значение|
|---|---|
|`<giftless-domain>`|Домен, на котором запущен Giftless|
|`<user_or_org>`|Имя владельца репозитория (GitHub user или организация)|
|`<repo>`|Название репозитория, обязательно с `.git` на конце для совместимости|

#### Пример:

```ini
[lfs]
url = https://giftless.example.com/my-org/my-repo.git/info/lfs
```

> [!note]  
> Обязательно оставь `.git/info/lfs` — это часть протокола Git LFS и **необходимо для работы**.

### 4. Настройка `git credential-store`

#### Включи хранилище токенов:

```bash
git config --global credential.helper store
```

#### Добавь PAT в `.git-credentials`:

```bash
git credential-store store <<EOF      
protocol=https
host=<giftless-domain>
username=git
password=<PAT>
EOF
```

#### Пример:

```bash
git credential-store store <<EOF      
protocol=https
host=giftless.example.com
username=git
password=ghp_abc1234exampletoken
EOF
```

> [!note] 
> `git:` — логин для BasicAuth (можно любой). Главное — **токен как пароль**.

### 5. Проверка

Отслеживаем и отправляем файл:

```bash
git lfs track "*.mp4"
git add .gitattributes
git add demo.mp4
git commit -m "Add demo file"
git push
```

Если всё настроено правильно — файл будет сохранён на стороне Giftless, а не в Git.

## Резюме

| Шаг                    | Действие                                                             |
| ---------------------- | -------------------------------------------------------------------- |
| Получить PAT           | Через [GitHub Settings](https://github.com/settings/tokens)          |
| Настроить `.lfsconfig` | Указать Giftless URL: `https://<domain>/<owner>/<repo>.git/info/lfs` |
| Сохранить токен        | `git config --global credential.helper store` + `~/.git-credentials` |
| Проверка               | `git lfs track`, `git push`                                          |
