# Руководство по разрешению конфликтов слияния

## 🎯 Общая стратегия

Когда у вас конфликты в `server/`, `drizzle/`, `src/` - нужно понимать что изменилось в обеих ветках.

---

## 📊 Диагностика перед слиянием

### 1. Посмотрите измененные файлы

```bash
# Список файлов с конфликтами
git diff --name-only main..other-branch

# Группировка по папкам
git diff --name-only main..other-branch | sort | uniq

# Статистика изменений
git diff --stat main..other-branch
```

### 2. Создайте список изменений

```bash
# Сохраните в файл для анализа
git diff --stat main..other-branch > changes.txt
git diff main..other-branch > full-diff.txt
```

---

## 🔧 Пошаговое слияние

### Шаг 1: Создайте ветку для слияния

```bash
# Убедитесь что всё закоммичено
git checkout main
git status

git checkout other-branch  
git status

# Создайте новую ветку
git checkout main
git checkout -b merge-branches
```

### Шаг 2: Попробуйте слить

```bash
git merge other-branch
```

Возможные сценарии:

#### Сценарий A: Нет конфликтов ✅
```
Auto-merging server/db.ts
Merge made by the 'recursive' strategy.
```
→ Всё хорошо! Переходите к тестированию.

#### Сценарий B: Есть конфликты ⚠️
```
Auto-merging server/routers.ts
CONFLICT (content): Merge conflict in server/routers.ts
Auto-merging server/drizzle/schema.ts
CONFLICT (content): Merge conflict in server/drizzle/schema.ts
Automatic merge failed; fix conflicts and then commit the result.
```
→ Нужно разрешить вручную.

---

## 🎨 Разрешение конфликтов

### Понимание маркеров конфликта

Когда Git находит конфликт, он помечает файл так:

```typescript
<<<<<<< HEAD (текущая ветка - main)
// Код из main
export const users = mysqlTable("users", {
  id: int("id").autoincrement().primaryKey(),
  email: varchar("email", { length: 320 }),
});
=======
// Код из other-branch
export const users = mysqlTable("users", {
  id: int("id").autoincrement().primaryKey(),
  email: varchar("email", { length: 320 }),
  phone: varchar("phone", { length: 20 }), // НОВОЕ ПОЛЕ
});
>>>>>>> other-branch
```

### Ваши действия:

1. **Удалите маркеры** (`<<<<<<<`, `=======`, `>>>>>>>`)
2. **Объедините изменения** из обеих веток
3. **Сохраните результат**

Правильное решение:
```typescript
export const users = mysqlTable("users", {
  id: int("id").autoincrement().primaryKey(),
  email: varchar("email", { length: 320 }),
  password_hash: varchar("password_hash", { length: 255 }), // из main
  phone: varchar("phone", { length: 20 }), // из other-branch
});
```

---

## 📁 Конфликты по типам файлов

### 1. `server/drizzle/schema.ts`

**Конфликт:** Обе ветки добавили поля в таблицы

**Решение:**
```typescript
// Объедините ВСЕ новые поля из обеих веток
export const users = mysqlTable("users", {
  // Существующие поля
  id: int("id").autoincrement().primaryKey(),
  openId: varchar("openId", { length: 64 }).notNull().unique(),
  name: text("name"),
  email: varchar("email", { length: 320 }),
  
  // ИЗ MAIN (auth ветка)
  password_hash: varchar("password_hash", { length: 255 }),
  
  // ИЗ OTHER-BRANCH (ваша новая ветка)
  phone: varchar("phone", { length: 20 }),
  avatar_url: text("avatar_url"),
  
  // Остальные поля...
  role: mysqlEnum("role", ["user", "admin"]).default("user").notNull(),
  createdAt: timestamp("createdAt").defaultNow().notNull(),
});

// ТАКЖЕ добавьте НОВЫЕ ТАБЛИЦЫ из обеих веток
export const loginAttempts = mysqlTable(...);  // из main
export const userProfiles = mysqlTable(...);   // из other-branch
```

### 2. `server/db.ts`

**Конфликт:** Обе ветки добавили функции

**Решение:**
```typescript
// Импорты из обеих веток
import { users, loginAttempts, userProfiles } from "../drizzle/schema";

// Функции из main
export async function getUserByEmail(email: string) { ... }
export async function createUserWithPassword(data: any) { ... }
export async function recordLoginAttempt(...) { ... }

// Функции из other-branch  
export async function getUserProfile(userId: number) { ... }
export async function updateUserProfile(...) { ... }

// Остальные функции...
```

### 3. `server/routers.ts`

**Конфликт:** Обе ветки добавили роутеры

**Решение:**
```typescript
export const appRouter = router({
  system: systemRouter,
  
  // Из main (auth)
  auth: router({
    me: publicProcedure.query(...),
    login: publicProcedure.mutation(...),
    logout: publicProcedure.mutation(...),
  }),
  
  // Из other-branch (ваши новые роутеры)
  profile: router({
    get: protectedProcedure.query(...),
    update: protectedProcedure.mutation(...),
  }),
  
  // Существующие роутеры
  device: router({ ... }),
  playlist: router({ ... }),
  // ...
});
```

### 4. `client/src/App.tsx`

**Конфликт:** Обе ветки добавили роуты

**Решение:**
```typescript
function Router() {
  return (
    <Switch>
      {/* Публичные роуты */}
      <Route path="/login" component={Login} />  {/* из main */}
      <Route path="/player" component={PlayerPage} />
      
      {/* Защищённые роуты */}
      <Route path="/">
        <AuthGuard>  {/* из main */}
          <Home />
        </AuthGuard>
      </Route>
      
      <Route path="/profile">  {/* из other-branch */}
        <AuthGuard>
          <Profile />
        </AuthGuard>
      </Route>
      
      <Route path="/devices">
        <AuthGuard>
          <Devices />
        </AuthGuard>
      </Route>
      
      {/* Остальные роуты... */}
    </Switch>
  );
}
```

### 5. `client/src/pages/*` (новые страницы)

**Конфликт:** Обычно нет, но могут быть импорты

**Решение:**
```typescript
// Если обе ветки изменили импорты
import { trpc } from "@/lib/trpc";
import { Button } from "@/components/ui/button";
import { AuthGuard } from "@/components/AuthGuard";  // из main
import { UserAvatar } from "@/components/UserAvatar";  // из other-branch
```

### 6. `package.json`

**Конфликт:** Обе ветки добавили зависимости

**Решение:**
```json
{
  "dependencies": {
    "react": "^18.0.0",
    // Из main
    "bcrypt": "^5.1.1",
    // Из other-branch
    "react-phone-input-2": "^2.15.1",
    "libphonenumber-js": "^1.10.0",
    // Остальное...
  },
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    // Из main
    "db:push": "drizzle-kit push",
    // Из other-branch
    "test": "vitest",
    // Остальное...
  }
}
```

После разрешения:
```bash
pnpm install  # Установить новые зависимости
```

---

## 🔄 Процесс разрешения конфликтов

### Метод 1: Вручную (рекомендуется для понимания)

```bash
# 1. Посмотрите список конфликтов
git status

# 2. Откройте каждый файл с конфликтом
code server/drizzle/schema.ts  # или nano/vim

# 3. Найдите маркеры <<<<<<< и разрешите
# 4. Удалите все маркеры
# 5. Сохраните файл

# 6. Добавьте разрешённый файл
git add server/drizzle/schema.ts

# 7. Повторите для всех конфликтов
git add server/db.ts
git add server/routers.ts
# ...

# 8. Закоммитьте слияние
git commit -m "Merge: combined auth branch with feature branch"
```

### Метод 2: Использовать merge tool

```bash
# Настройте merge tool (один раз)
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'

# Или используйте встроенный vimdiff
git config --global merge.tool vimdiff

# Запустите merge tool
git mergetool

# Он откроет каждый конфликт в визуальном редакторе
# Выберите какую версию оставить или объедините вручную
```

### Метод 3: Выбрать одну версию полностью

Если уверены что нужна только одна версия файла:

```bash
# Оставить версию из main
git checkout --ours server/some-file.ts
git add server/some-file.ts

# Или оставить версию из other-branch
git checkout --theirs server/some-file.ts
git add server/some-file.ts
```

⚠️ **ОСТОРОЖНО:** Это удалит изменения из другой ветки!

---

## ✅ После разрешения всех конфликтов

### 1. Проверка

```bash
# Убедитесь что все конфликты разрешены
git status

# Должно быть:
# All conflicts fixed but you are still merging.
# (use "git commit" to conclude merge)
```

### 2. Тестирование перед коммитом

```bash
# Установите зависимости (если package.json изменился)
pnpm install

# Пересоберите
pnpm build

# Если есть ошибки компиляции - исправьте!
```

### 3. Обновите миграции БД

```bash
# Сгенерируйте новую миграцию для объединённой схемы
pnpm db:generate

# Проверьте что миграция правильная
cat drizzle/migrations/XXXX_*.sql
```

### 4. Закоммитьте

```bash
git commit -m "Merge: auth + feature branch

- Added password authentication (main)
- Added user profiles (other-branch)
- Resolved conflicts in schema.ts, db.ts, routers.ts
- Updated migrations
- All tests passing"
```

### 5. Финальное тестирование

```bash
# Запустите в dev режиме
pnpm dev

# Проверьте:
# ✅ Логин работает
# ✅ Новые функции работают
# ✅ Старые функции не сломались
# ✅ БД схема правильная
```

---

## 🚨 Если что-то пошло не так

### Отменить слияние

```bash
# Если ещё не закоммитили
git merge --abort

# Если уже закоммитили но поняли что ошибка
git reset --hard HEAD~1

# Или вернуться к последнему рабочему коммиту
git reflog  # найдите hash рабочего коммита
git reset --hard <commit-hash>
```

### Начать заново

```bash
# Удалите ветку слияния
git checkout main
git branch -D merge-branches

# Создайте новую и попробуйте снова
git checkout -b merge-branches-v2
git merge other-branch
```

---

## 📝 Чеклист перед слиянием в main

- [ ] Все конфликты разрешены
- [ ] `git status` чистый
- [ ] `pnpm build` успешен
- [ ] Все новые зависимости установлены
- [ ] Миграции БД сгенерированы и проверены
- [ ] Приложение запускается (`pnpm dev`)
- [ ] Логин работает
- [ ] Новые фичи работают
- [ ] Старые фичи не сломались
- [ ] Нет console.error в браузере
- [ ] Проверили на нескольких устройствах

---

## 🎯 Специфичные конфликты для вашего проекта

### Конфликт: Добавление полей в `users`

```typescript
// MAIN добавил:
password_hash: varchar("password_hash", { length: 255 }),

// OTHER-BRANCH добавил:
phone: varchar("phone", { length: 20 }),

// РЕШЕНИЕ - оставьте оба:
password_hash: varchar("password_hash", { length: 255 }),
phone: varchar("phone", { length: 20 }),
```

### Конфликт: Новые таблицы

```typescript
// MAIN добавил:
export const loginAttempts = mysqlTable("login_attempts", { ... });

// OTHER-BRANCH добавил:
export const userProfiles = mysqlTable("user_profiles", { ... });

// РЕШЕНИЕ - оставьте обе таблицы:
export const loginAttempts = mysqlTable("login_attempts", { ... });
export const userProfiles = mysqlTable("user_profiles", { ... });
```

### Конфликт: Новые функции в db.ts

```typescript
// MAIN добавил:
export async function getUserByEmail(email: string) { ... }

// OTHER-BRANCH добавил:  
export async function getUserByPhone(phone: string) { ... }

// РЕШЕНИЕ - оставьте обе функции
```

### Конфликт: Импорты

```typescript
// MAIN:
import { users, loginAttempts } from "../drizzle/schema";

// OTHER-BRANCH:
import { users, userProfiles } from "../drizzle/schema";

// РЕШЕНИЕ - объедините:
import { users, loginAttempts, userProfiles } from "../drizzle/schema";
```

---

## 💡 Лучшие практики

1. **Коммитьте часто** - маленькие коммиты легче сливать
2. **Sync регулярно** - `git pull origin main` в вашу ветку
3. **Тестируйте перед merge** - убедитесь что ветка работает
4. **Документируйте изменения** - комментарии в коммитах
5. **Бэкап перед merge** - `git branch backup-before-merge`

---

## 🆘 Когда просить помощь

Попросите другого разработчика (или меня 😊) если:

- Больше 10 файлов с конфликтами
- Конфликты в критических файлах (auth, БД)
- Не уверены какую версию оставить
- После слияния что-то сломалось

**Совет:** Покажите мне список конфликтных файлов (`git diff --name-only main..other-branch`) и я помогу определить стратегию!

---

## 📚 Дополнительные ресурсы

```bash
# Хорошая визуализация истории
git log --graph --oneline --all

# Сравнение веток
git diff main...other-branch

# Найти общего предка
git merge-base main other-branch

# История изменений файла
git log --follow --all -- server/drizzle/schema.ts
```
