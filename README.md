# Клонирование репозитория с GitHub на Ubuntu/Debian

## Предварительная установка

Установите Git, если он еще не установлен:
```bash
sudo apt update
sudo apt install git
```

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

### Добавление ключа в SSH-агент
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_github
```

### Копирование публичного ключа
```bash
cat ~/.ssh/id_ed25519_github.pub
```
Скопируйте вывод и добавьте в GitHub: Settings → SSH and GPG keys → New SSH key

### Настройка SSH конфига для нескольких аккаунтов

Создайте/отредактируйте файл `~/.ssh/config`:
```bash
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

## 7. Переключение между пользователями в существующем репозитории

```bash
# Изменить email и имя для конкретного репозитория
git config user.name "Другое Имя"
git config user.email "other.email@example.com"

# Изменить URL удаленного репозитория
git remote set-url origin git@github.com-work:username/repo.git
```

## 8. Полезные команды

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
```

## Распространенные проблемы

**Permission denied (publickey)** - SSH ключ не настроен или не добавлен на GitHub

**Authentication failed** - Неверные credentials для HTTPS или токен истек

**fatal: not a git repository** - Вы не находитесь в директории Git репозитория

## Рекомендации

- Используйте SSH для приватных репозиториев и постоянной работы
- Используйте локальную конфигурацию `git config` (без `--global`) для проектов с разными аккаунтами
- Регулярно обновляйте SSH ключи и используйте разные ключи для разных аккаунтов
- Для работы с Personal Access Token вместо пароля создайте токен в GitHub Settings → Developer settings
