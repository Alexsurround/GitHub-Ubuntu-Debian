# Клонирование репозитория с GitHub на Ubuntu/Debian и Windows

## Предварительная установка

### Linux (Ubuntu/Debian)
Установите Git, если он еще не установлен:
```bash
sudo apt update
sudo apt install git
```

### Windows
1. Скачайте Git for Windows с официального сайта: https://git-scm.com/download/win
2. Запустите установщик
3. Рекомендуемые настройки при установке:
   - Выберите "Use Git from Git Bash only" или "Git from the command line and also from 3rd-party software"
   - Выберите "Use bundled OpenSSH"
   - Выберите "Use the OpenSSL library"
   - Выберите "Checkout Windows-style, commit Unix-style line endings"
4. После установки откройте **Git Bash** из меню Пуск

## 1. Клонирование репозитория

### Через HTTPS (для публичных репозиториев)
```bash
git clone https://github.com/username/repository.git
cd repository
```

### Через SSH (рекомендуется для приватных репозиториев)
```bash
git clone git@github.com:username/repository.git
cd repository
```

## 2. Настройка пользователя Git

### Глобальная конфигурация (для всех репозиториев)
```bash
git config --global user.name "Ваше Имя"
git config --global user.email "your.email@example.com"
```

### Локальная конфигурация (только для текущего репозитория)
```bash
git config user.name "Ваше Имя"
git config user.email "your.email@example.com"
```

## 3. Настройка SSH ключей для разных пользователей

### Генерация SSH ключа
```bash
ssh-keygen -t ed25519 -C "your.email@example.com" -f ~/.ssh/id_ed25519_github
```

При появлении запроса введите пароль для дополнительной безопасности (опционально).

### Добавление ключа в SSH-агент
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_github
```

### Копирование публичного ключа
```bash
cat ~/.ssh/id_ed25519_github.pub
```

### Добавление SSH ключа в GitHub

1. Скопируйте вывод команды выше (весь текст, начинающийся с `ssh-ed25519`)
2. Откройте GitHub в браузере и войдите в свой аккаунт
3. Нажмите на свой аватар → **Settings**
4. В боковом меню выберите **SSH and GPG keys**
5. Нажмите **New SSH key**
6. Заполните форму:
   - **Title**: Описательное имя (например, "Ubuntu Laptop Work")
   - **Key type**: Authentication Key
   - **Key**: Вставьте скопированный публичный ключ
7. Нажмите **Add SSH key**
8. Подтвердите действие паролем GitHub при необходимости

### Проверка SSH подключения
```bash
ssh -T git@github.com
```
Вы должны увидеть: `Hi username! You've successfully authenticated...`

### Настройка SSH конфига для нескольких аккаунтов

**Linux:**
```bash
nano ~/.ssh/config
```

**Windows (Git Bash):**
```bash
notepad ~/.ssh/config
# или
nano ~/.ssh/config
```

Добавьте конфигурацию:
```
# Личный аккаунт
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal

# Рабочий аккаунт
Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
```

## 4. Клонирование с использованием разных аккаунтов

```bash
# Для личного аккаунта
git clone git@github.com-personal:username/repo.git

# Для рабочего аккаунта
git clone git@github.com-work:username/repo.git
```

## 5. Настройка credential helper для HTTPS

Для сохранения учетных данных при работе через HTTPS:
```bash
git config --global credential.helper store
```

Или использовать кэширование на 1 час:
```bash
git config --global credential.helper 'cache --timeout=3600'
```

## 6. Проверка конфигурации

```bash
# Просмотр текущей конфигурации
git config --list

# Проверка подключения SSH
ssh -T git@github.com

# Проверка удаленных репозиториев
git remote -v
```

## 7. Настройка GPG ключей для подписи коммитов

### Установка GPG

**Linux (Ubuntu/Debian):**
```bash
sudo apt install gnupg
```

**Windows:**
GPG уже входит в состав Git for Windows. Проверьте:
```bash
gpg --version
```

Если GPG отсутствует, установите Gpg4win: https://www.gpg4win.org/

### Генерация GPG ключа
```bash
gpg --full-generate-key
```

Выберите:
- Тип ключа: `(1) RSA and RSA` (по умолчанию)
- Размер ключа: `4096`
- Срок действия: `0` (не истекает) или укажите срок
- Введите имя и email (должен совпадать с email в Git)

**Примечание для Windows:** При генерации ключа может открыться окно для ввода пароля (passphrase).

### Просмотр списка GPG ключей
```bash
gpg --list-secret-keys --keyid-format=long
```

Вывод будет примерно таким:
```
sec   rsa4096/3AA5C34371567BD2 2024-01-01 [SC]
      ABC123DEF456GHI789JKL012MNO345PQR678STU
uid                 [ultimate] Your Name <your.email@example.com>
ssb   rsa4096/42B317FD4BA89E7A 2024-01-01 [E]
```

Запомните ID ключа после `rsa4096/` (в примере: `3AA5C34371567BD2`)

### Экспорт публичного GPG ключа
```bash
gpg --armor --export 3AA5C34371567BD2
```

### Добавление GPG ключа в GitHub

1. Скопируйте вывод команды выше (весь блок от `-----BEGIN PGP PUBLIC KEY BLOCK-----` до `-----END PGP PUBLIC KEY BLOCK-----`)
2. Откройте GitHub → **Settings** → **SSH and GPG keys**
3. Нажмите **New GPG key**
4. Вставьте скопированный ключ в поле **Key**
5. Нажмите **Add GPG key**

### Настройка Git для использования GPG

```bash
# Указать GPG ключ для подписи
git config --global user.signingkey 3AA5C34371567BD2

# Автоматически подписывать все коммиты
git config --global commit.gpgsign true

# Автоматически подписывать все теги
git config --global tag.gpgsign true
```

### Настройка GPG для работы в терминале (если возникают проблемы)

**Linux:**
```bash
echo 'export GPG_TTY=$(tty)' >> ~/.bashrc
source ~/.bashrc
```

**Windows (Git Bash):**
Добавьте в `~/.bash_profile` или `~/.bashrc`:
```bash
echo 'export GPG_TTY=$(tty)' >> ~/.bash_profile
source ~/.bash_profile
```

Для Windows также может потребоваться:
```bash
git config --global gpg.program "C:/Program Files/Git/usr/bin/gpg.exe"
```

### Создание подписанного коммита
```bash
# Автоматически (если включено commit.gpgsign)
git commit -m "Your commit message"

# Вручную с флагом -S
git commit -S -m "Your commit message"
```

### Проверка подписи коммита
```bash
git log --show-signature
```

На GitHub подписанные коммиты будут отображаться с зеленой меткой "Verified".

## 8. Переключение между пользователями в существующем репозитории

```bash
# Изменить email и имя для конкретного репозитория
git config user.name "Другое Имя"
git config user.email "other.email@example.com"

# Изменить URL удаленного репозитория
git remote set-url origin git@github.com-work:username/repo.git
```

## 9. Полезные команды

```bash
# Клонирование конкретной ветки
git clone -b branch-name https://github.com/username/repo.git

# Клонирование с ограниченной историей (быстрее)
git clone --depth 1 https://github.com/username/repo.git

# Просмотр текущего пользователя
git config user.name
git config user.email

# Удаление сохраненных credentials
git config --global --unset credential.helper

# Проверка GPG ключей
gpg --list-keys

# Экспорт приватного GPG ключа (для резервного копирования)
gpg --armor --export-secret-keys your.email@example.com > private-key.asc

# Импорт GPG ключа на другой машине
gpg --import private-key.asc
```

## Распространенные проблемы

**Permission denied (publickey)** - SSH ключ не настроен или не добавлен на GitHub

**Authentication failed** - Неверные credentials для HTTPS или токен истек

**fatal: not a git repository** - Вы не находитесь в директории Git репозитория

**gpg: signing failed: Inappropriate ioctl for device** - Добавьте `export GPG_TTY=$(tty)` в ~/.bashrc (Linux) или ~/.bash_profile (Windows)

**error: gpg failed to sign the data** - Проверьте, что GPG ключ настроен правильно: `git config user.signingkey`

**Windows: ssh-add не работает** - Запустите `eval $(ssh-agent -s)` перед добавлением ключа

**Windows: проблемы с правами на файлы SSH** - Выполните:
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519_github
chmod 644 ~/.ssh/id_ed25519_github.pub
```

**Windows: проблемы с переносами строк (CRLF)** - Настройте:
```bash
git config --global core.autocrlf true
```

## Рекомендации

- Используйте SSH для приватных репозиториев и постоянной работы
- Используйте локальную конфигурацию `git config` (без `--global`) для проектов с разными аккаунтами
- Регулярно обновляйте SSH ключи и используйте разные ключи для разных аккаунтов
- Используйте GPG подпись для важных проектов и верификации авторства
- Храните резервные копии приватных GPG ключей в безопасном месте
- Для работы с Personal Access Token вместо пароля создайте токен в GitHub Settings → Developer settings
- На GitHub в разделе **Settings → SSH and GPG keys** вы можете управлять всеми своими ключами и видеть, когда они последний раз использовались

## Особенности работы в Windows Git Bash

### Пути к файлам
- Домашняя директория: `/c/Users/YourUsername/` или `~`
- SSH ключи находятся в: `~/.ssh/`
- Git конфигурация: `~/.gitconfig`

### Открытие Git Bash
- Через меню Пуск: "Git Bash"
- Из контекстного меню: правый клик в папке → "Git Bash Here"
- Из VS Code: встроенный терминал с профилем Git Bash

### Редакторы в Git Bash
```bash
# Использование Notepad
notepad ~/.ssh/config

# Использование nano (встроен в Git Bash)
nano ~/.ssh/config

# Использование VS Code (если установлен)
code ~/.ssh/config

# Установка VS Code как редактор по умолчанию для Git
git config --global core.editor "code --wait"
```

### Копирование в буфер обмена (Windows)
```bash
# Скопировать SSH ключ
cat ~/.ssh/id_ed25519_github.pub | clip

# Скопировать GPG ключ
gpg --armor --export 3AA5C34371567BD2 | clip
```

### Интеграция с Windows Credential Manager
```bash
# Использовать Windows Credential Manager для хранения паролей
git config --global credential.helper manager
```
