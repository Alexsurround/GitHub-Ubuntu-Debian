# Полная шпаргалка по Git, GitHub и SSH

## Содержание
1. [Установка Git](#установка-git)
2. [Клонирование репозитория](#клонирование-репозитория)
3. [Настройка пользователя](#настройка-пользователя)
4. [Настройка SSH ключей](#настройка-ssh-ключей)
5. [Импорт SSH ключей](#импорт-ssh-ключей)
6. [Настройка GPG ключей](#настройка-gpg-ключей)
7. [Работа с ветками](#работа-с-ветками)
8. [Синхронизация с удаленным репозиторием](#синхронизация-с-удаленным-репозиторием)
9. [Решение конфликтов](#решение-конфликтов)
10. [Отмена изменений](#отмена-изменений)
11. [Windows Git Bash](#windows-git-bash)
12. [Полезные команды](#полезные-команды)

---

## Установка Git

### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install git
```

### Windows
1. Скачать: https://git-scm.com/download/win
2. Установить с рекомендуемыми настройками
3. Открыть Git Bash

### Проверка установки
```bash
git --version
```

---

## Клонирование репозитория

### HTTPS (публичные репозитории)
```bash
git clone https://github.com/username/repository.git
cd repository
```

### SSH (приватные репозитории)
```bash
git clone git@github.com:username/repository.git
cd repository
```

### Клонирование конкретной ветки
```bash
git clone -b branch-name https://github.com/username/repo.git
```

### Клонирование с ограниченной историей
```bash
git clone --depth 1 https://github.com/username/repo.git
```

---

## Настройка пользователя

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

### Просмотр конфигурации
```bash
git config --list
git config user.name
git config user.email
```

---

## Настройка SSH ключей

### Генерация SSH ключа
```bash
ssh-keygen -t ed25519 -C "your.email@example.com" -f ~/.ssh/id_ed25519_github
```

### Добавление ключа в SSH-агент

**Linux:**
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_github
```

**Windows Git Bash:**
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519_github
```

### Копирование публичного ключа

**Linux:**
```bash
cat ~/.ssh/id_ed25519_github.pub
```

**Windows Git Bash:**
```bash
cat ~/.ssh/id_ed25519_github.pub | clip
```

### Добавление SSH ключа на GitHub

1. Открыть GitHub → Settings → SSH and GPG keys
2. Нажать **New SSH key**
3. Title: "Ubuntu Laptop" или "Windows PC"
4. Key: Вставить скопированный ключ
5. Нажать **Add SSH key**

### Проверка SSH подключения
```bash
ssh -T git@github.com
# Должно вывести: Hi username! You've successfully authenticated...
```

### SSH config для нескольких аккаунтов

**Linux:**
```bash
nano ~/.ssh/config
```

**Windows:**
```bash
notepad ~/.ssh/config
```

**Содержимое:**
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

**Клонирование с разными аккаунтами:**
```bash
git clone git@github.com-personal:username/repo.git
git clone git@github.com-work:username/repo.git
```

---

## Импорт SSH ключей

### Импорт из резервной копии (общий способ)
```bash
# Создать директорию .ssh если её нет
mkdir -p ~/.ssh

# Скопировать файлы ключей
cp /path/to/id_ed25519 ~/.ssh/
cp /path/to/id_ed25519.pub ~/.ssh/

# Установить правильные права доступа (КРИТИЧЕСКИ ВАЖНО!)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# Добавить ключ в SSH-агент
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Проверить что ключ добавлен
ssh-add -l

# Проверить подключение
ssh -T git@github.com
```

### Импорт через SCP из Windows в Linux

**На Linux узнать IP адрес:**
```bash
hostname -I
# или
ip addr show
```

**На Linux установить SSH сервер (если не установлен):**
```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh
```

**На Windows Git Bash скопировать ключи:**
```bash
# Вариант 1: Скопировать в home директорию
scp ~/.ssh/id_ed25519 username@192.168.1.100:~/
scp ~/.ssh/id_ed25519.pub username@192.168.1.100:~/

# Вариант 2: Скопировать напрямую в .ssh (рекомендуется)
scp ~/.ssh/id_ed25519 username@192.168.1.100:~/.ssh/
scp ~/.ssh/id_ed25519.pub username@192.168.1.100:~/.ssh/

# Скопировать всю папку .ssh
scp -r ~/.ssh username@192.168.1.100:~/ssh_backup
```

**На Linux после копирования:**
```bash
# Если скопировали в home
mkdir -p ~/.ssh
mv ~/id_ed25519 ~/.ssh/
mv ~/id_ed25519.pub ~/.ssh/

# Если скопировали всю папку
mv ~/ssh_backup ~/.ssh

# Установить права
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub

# Добавить в SSH-агент
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Проверить
ssh -T git@github.com
```

### Импорт через WSL
```bash
# В WSL Linux
cp /mnt/c/Users/YourUsername/.ssh/id_ed25519 ~/.ssh/
cp /mnt/c/Users/YourUsername/.ssh/id_ed25519.pub ~/.ssh/

chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
ssh-add ~/.ssh/id_ed25519
```

### Импорт с другого Linux компьютера
```bash
# С удалённого компьютера на локальный
scp user@remote-host:~/.ssh/id_ed25519 ~/.ssh/
scp user@remote-host:~/.ssh/id_ed25519.pub ~/.ssh/

chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
ssh-add ~/.ssh/id_ed25519
```

---

## Настройка GPG ключей

### Установка GPG

**Linux:**
```bash
sudo apt install gnupg
```

**Windows:**
GPG входит в Git for Windows. Проверка:
```bash
gpg --version
```

### Генерация GPG ключа
```bash
gpg --full-generate-key

# Выбрать:
# - Тип: (1) RSA and RSA
# - Размер: 4096
# - Срок: 0 (не истекает)
# - Ввести имя и email
```

### Просмотр GPG ключей
```bash
gpg --list-secret-keys --keyid-format=long

# Вывод:
# sec   rsa4096/3AA5C34371567BD2 2024-01-01 [SC]
# Запомнить ID: 3AA5C34371567BD2
```

### Экспорт публичного GPG ключа
```bash
gpg --armor --export 3AA5C34371567BD2
```

**Windows (копирование в буфер):**
```bash
gpg --armor --export 3AA5C34371567BD2 | clip
```

### Добавление GPG ключа на GitHub

1. GitHub → Settings → SSH and GPG keys
2. Нажать **New GPG key**
3. Вставить скопированный ключ
4. Нажать **Add GPG key**

### Настройка Git для использования GPG
```bash
# Указать GPG ключ для подписи
git config --global user.signingkey 3AA5C34371567BD2

# Автоматически подписывать все коммиты
git config --global commit.gpgsign true

# Автоматически подписывать все теги
git config --global tag.gpgsign true
```

### Настройка GPG для терминала

**Linux:**
```bash
echo 'export GPG_TTY=$(tty)' >> ~/.bashrc
source ~/.bashrc
```

**Windows Git Bash:**
```bash
echo 'export GPG_TTY=$(tty)' >> ~/.bash_profile
source ~/.bash_profile

# Указать путь к gpg
git config --global gpg.program "C:/Program Files/Git/usr/bin/gpg.exe"
```

### Создание подписанного коммита
```bash
# Автоматически (если включено commit.gpgsign)
git commit -m "Your commit message"

# Вручную с флагом -S
git commit -S -m "Your commit message"
```

### Проверка подписи
```bash
git log --show-signature
```

### Экспорт/импорт GPG ключей
```bash
# Экспорт приватного ключа (резервная копия)
gpg --armor --export-secret-keys your.email@example.com > private-key.asc

# Импорт на другой машине
gpg --import private-key.asc

# Список ключей
gpg --list-keys
```

---

## Работа с ветками

### Просмотр веток
```bash
# Локальные ветки
git branch

# Все ветки (включая удалённые)
git branch -a

# Удалённые ветки
git branch -r
```

### Создание и переключение
```bash
# Создать новую ветку
git branch feature-name

# Переключиться на ветку
git checkout feature-name

# Создать и сразу переключиться
git checkout -b feature-name

# Современный синтаксис
git switch feature-name
git switch -c feature-name
```

### Слияние веток
```bash
# Переключиться на main
git checkout main

# Слить ветку feature-name в main
git merge feature-name
```

### Удаление веток
```bash
# Удалить локальную ветку
git branch -d feature-name

# Принудительное удаление
git branch -D feature-name

# Удалить удалённую ветку
git push origin --delete feature-name
```

---

## Синхронизация с удаленным репозиторием

### Базовые команды

```bash
# Получить изменения с GitHub (без слияния)
git fetch origin

# Получить и слить изменения
git pull origin main

# Отправить изменения на GitHub
git push origin main

# Установить ветку по умолчанию для push
git push -u origin main
```

### Статус синхронизации
```bash
# Проверить статус
git status

# Сравнить с удалённой веткой
git fetch origin
git log HEAD..origin/main --oneline    # Что есть на GitHub
git log origin/main..HEAD --oneline    # Что есть локально
```

### Различные сценарии

**Локальные изменения не закоммичены:**
```bash
# Вариант 1: Сохранить и отправить
git add .
git commit -m "Описание изменений"
git push origin main

# Вариант 2: Отложить изменения
git stash save "Описание"
git pull origin main
git stash pop

# Вариант 3: Отменить изменения
git restore .
```

**Локальные коммиты не отправлены:**
```bash
git push origin main
```

**Удалённый репозиторий впереди:**
```bash
# Простое слияние
git pull origin main

# С rebase (более чистая история)
git pull --rebase origin main
```

**Расхождение истории (diverged):**
```bash
# Вариант 1: Merge
git fetch origin
git merge origin/main
git push origin main

# Вариант 2: Rebase
git fetch origin
git rebase origin/main
git push origin main

# Вариант 3: Создать ветку для слияния
git checkout -b merge-branch
git merge origin/main
# Решить конфликты
git push origin merge-branch
# Создать Pull Request на GitHub
```

---

## Решение конфликтов

### Определить конфликтующие файлы
```bash
git status

# Вывод покажет:
# Unmerged paths:
#   both modified:   file.txt
```

### Просмотр конфликта
Файл будет содержать маркеры:
```
<<<<<<< HEAD
Ваша локальная версия кода
=======
Версия кода из GitHub
>>>>>>> origin/main
```

### Решение конфликта
```bash
# 1. Отредактировать файл (удалить маркеры, выбрать нужный код)

# 2. Отметить конфликт как решённый
git add file.txt

# 3. Проверить статус
git status

# 4. Завершить слияние
git commit -m "Resolve merge conflict in file.txt"

# 5. Отправить на GitHub
git push origin main
```

### Отмена слияния
```bash
# Отменить merge
git merge --abort

# Отменить rebase
git rebase --abort
```

### Использование mergetool
```bash
# Запустить визуальный инструмент
git mergetool

# Настроить VS Code как mergetool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

---

## Отмена изменений

### Отмена незакоммиченных изменений
```bash
# Отменить изменения в конкретном файле
git restore file.txt
git checkout -- file.txt  # Старый синтаксис

# Отменить все изменения
git restore .
git checkout -- .

# Удалить неотслеживаемые файлы
git clean -fd
```

### Отмена последнего коммита
```bash
# Отменить коммит, сохранив изменения
git reset --soft HEAD~1

# Отменить коммит и staging, сохранив файлы
git reset HEAD~1

# ОСТОРОЖНО: Полностью удалить коммит и изменения
git reset --hard HEAD~1
```

### Отмена коммита на GitHub
```bash
# Создать новый коммит, отменяющий изменения
git revert HEAD
git push origin main

# Отменить конкретный коммит
git revert <commit-hash>
```

### Откат к конкретному коммиту
```bash
# Посмотреть историю
git log --oneline

# Вернуться к коммиту (сохранив изменения)
git reset --soft <commit-hash>

# ОСТОРОЖНО: Жёсткий откат
git reset --hard <commit-hash>

# Force push (если уже на GitHub)
git push --force-with-lease origin main
```

### Отмена изменений с сервера
```bash
# ОСТОРОЖНО: Заменить локальную версию версией с GitHub
git fetch origin
git reset --hard origin/main
git clean -fd
```

---

## Windows Git Bash

### Пути к файлам
```bash
# Домашняя директория
cd ~
cd /c/Users/YourUsername/

# SSH ключи
ls ~/.ssh/

# Git конфигурация
cat ~/.gitconfig
```

### Открытие Git Bash
- Меню Пуск → "Git Bash"
- Правый клик в папке → "Git Bash Here"
- VS Code → встроенный терминал

### Редакторы
```bash
# Notepad
notepad ~/.ssh/config

# Nano
nano ~/.ssh/config

# VS Code
code ~/.ssh/config

# Установить VS Code как редактор по умолчанию
git config --global core.editor "code --wait"
```

### Копирование в буфер обмена
```bash
# SSH ключ
cat ~/.ssh/id_ed25519.pub | clip

# GPG ключ
gpg --armor --export 3AA5C34371567BD2 | clip
```

### Автозапуск SSH-агента

**Создать/отредактировать ~/.bash_profile:**
```bash
nano ~/.bash_profile
```

**Добавить:**
```bash
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add ~/.ssh/id_ed25519
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add ~/.ssh/id_ed25519
fi

unset env
```

### Настройки для Windows
```bash
# Переносы строк (CRLF)
git config --global core.autocrlf true

# Credential Manager
git config --global credential.helper manager

# Права доступа к SSH файлам
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

---

## Полезные команды

### Просмотр различий
```bash
# Изменения в незакоммиченных файлах
git diff

# Изменения в staged файлах
git diff --staged

# Сравнить с удалённой веткой
git diff origin/main

# Сравнить два коммита
git diff <commit1> <commit2>

# Изменения в конкретном файле
git diff file.txt
```

### История коммитов
```bash
# Красивый вывод
git log --oneline --graph --all --decorate

# Подробная история
git log -p

# История конкретного файла
git log --follow file.txt

# Поиск по коммитам
git log --grep="ключевое слово"

# Кто изменял файл
git blame file.txt

# Последние N коммитов
git log -n 5
```

### Работа с удалёнными репозиториями
```bash
# Показать удалённые репозитории
git remote -v

# Добавить новый удалённый репозиторий
git remote add upstream https://github.com/original/repo.git

# Изменить URL
git remote set-url origin git@github.com:username/repo.git

# Удалить удалённый репозиторий
git remote remove origin

# Получить изменения со всех удалённых
git fetch --all
```

### Stash (временное хранилище)
```bash
# Сохранить изменения
git stash save "Описание изменений"

# Список сохранённых изменений
git stash list

# Применить последние изменения
git stash pop

# Применить конкретный stash
git stash apply stash@{0}

# Удалить stash
git stash drop stash@{0}

# Очистить все stash
git stash clear
```

### Теги
```bash
# Создать тег
git tag v1.0.0

# Создать аннотированный тег
git tag -a v1.0.0 -m "Версия 1.0.0"

# Список тегов
git tag

# Отправить тег на GitHub
git push origin v1.0.0

# Отправить все теги
git push origin --tags

# Удалить тег локально
git tag -d v1.0.0

# Удалить тег на GitHub
git push origin --delete v1.0.0
```

### Другие полезные команды
```bash
# Показать изменённые файлы
git status -s

# Показать последний коммит
git show

# Показать конкретный коммит
git show <commit-hash>

# Восстановление удалённого файла
git checkout HEAD -- file.txt

# История команд Git
git reflog

# Очистка репозитория
git gc

# Проверка целостности
git fsck
```

---

## Быстрая шпаргалка команд

### Ежедневная работа
```bash
git status                    # Проверить статус
git add .                     # Добавить все изменения
git commit -m "message"       # Создать коммит
git push origin main          # Отправить на GitHub
git pull origin main          # Получить с GitHub
```

### Работа с ветками
```bash
git branch                    # Список веток
git checkout -b feature       # Создать и переключиться
git merge feature             # Слить ветку
git branch -d feature         # Удалить ветку
```

### Отмена изменений
```bash
git restore file.txt          # Отменить изменения в файле
git restore .                 # Отменить все изменения
git reset HEAD~1              # Отменить последний коммит
git revert HEAD               # Отменить коммит (безопасно)
```

### SSH
```bash
ssh-keygen -t ed25519         # Создать SSH ключ
ssh-add ~/.ssh/id_ed25519     # Добавить в агент
ssh -T git@github.com         # Проверить подключение
```

### Импорт ключей
```bash
mkdir -p ~/.ssh               # Создать директорию
cp /path/key ~/.ssh/          # Скопировать ключ
chmod 600 ~/.ssh/id_ed25519   # Установить права
ssh-add ~/.ssh/id_ed25519     # Добавить в агент
```

### SCP (Windows → Linux)
```bash
scp ~/.ssh/id_ed25519 user@IP:~/.ssh/     # Копировать ключ
chmod 600 ~/.ssh/id_ed25519               # Установить права
ssh-add ~/.ssh/id_ed25519                 # Добавить в агент
```

---

## Решение проблем

### Permission denied (publickey)
```bash
ssh-add -l                    # Проверить ключи в агенте
ssh-add ~/.ssh/id_ed25519     # Добавить ключ
ssh -T git@github.com         # Проверить подключение
```

### Could not open a connection to your authentication agent
```bash
eval $(ssh-agent -s)          # Запустить агент
ssh-add ~/.ssh/id_ed25519     # Добавить ключ
```

### gpg failed to sign the data
```bash
export GPG_TTY=$(tty)         # Установить GPG_TTY
git config user.signingkey    # Проверить ключ
```

### Проблемы с правами (Linux)
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 644 ~/.ssh/config
```

### Конфликт при pull
```bash
git stash                     # Отложить изменения
git pull origin main          # Получить изменения
git stash pop                 # Вернуть изменения
```

### Сброс к состоянию на GitHub
```bash
git fetch origin
git reset --hard origin/main
git clean -fd
```

---

## Полезные ссылки

- **Git документация**: https://git-scm.com/doc
- **GitHub Docs**: https://docs.github.com/
- **GitLab Docs**: https://docs.gitlab.com/
- **Интерактивное обучение**: https://learngitbranching.js.org/
- **Git Cheat Sheet**: https://education.github.com/git-cheat-sheet-education.pdf

---

**Сохраните эту шпаргалку и используйте при работе с Git!** 🚀
