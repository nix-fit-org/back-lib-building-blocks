# 🚀 Инструкции по запуску BuildingBlocks

Этот файл содержит пошаговые инструкции для публикации BuildingBlocks пакетов на GitHub Packages.

---

## ✅ Что уже готово

В папке `C:\Mine\back-lib-building-blocks\` подготовлена полная структура:

```
back-lib-building-blocks/
├── .github/workflows/publish.yml    ✅ CI/CD настроен
├── .gitignore                       ✅ Git ignore готов
├── Directory.Packages.props         ✅ Версии пакетов
├── README.md                        ✅ Документация
└── src/
    ├── Nix.BuildingBlocks/          ✅ DDD building blocks
    ├── Nix.Messaging/               ✅ Event bus
    ├── Nix.Persistence/             ✅ Repository & UoW
    └── Nix.Contracts/               ✅ Integration events
```

---

## 📋 Следующие шаги

### Шаг 1: Создайте репозиторий на GitHub (5 минут)

1. Перейдите на GitHub: https://github.com/organizations/Nix/repositories/new
   
   **ИЛИ** если Nix это ваш личный аккаунт:
   https://github.com/new

2. Заполните:
   - **Repository name**: `back-lib-building-blocks`
   - **Description**: `Shared building blocks and contracts for FitCourse microservices`
   - **Visibility**: `Private` (рекомендуется) или `Public`
   - **Initialize this repository**: ❌ **НЕ ДОБАВЛЯЙТЕ** README, .gitignore, license

3. Нажмите **Create repository**

4. **Скопируйте URL репозитория**, например:
   ```
   https://github.com/nix-fit-org/back-lib-building-blocks.git
   ```

---

### Шаг 2: Инициализируйте Git и push (5 минут)

Откройте PowerShell и выполните:

```powershell
# Перейдите в папку
cd C:\Mine\back-lib-building-blocks

# Инициализируйте Git
git init

# Добавьте все файлы
git add .

# Сделайте первый коммит
git commit -m "Initial commit: BuildingBlocks, Messaging, Persistence, Contracts"

# Укажите URL удаленного репозитория (ЗАМЕНИТЕ на ваш URL!)
git remote add origin https://github.com/nix-fit-org/back-lib-building-blocks.git

# Переименуйте ветку в main (если нужно)
git branch -M main

# Push в GitHub
git push -u origin main
```

**⚠️ ВАЖНО:** Замените `https://github.com/nix-fit-org/back-lib-building-blocks.git` на ваш реальный URL!

---

### Шаг 3: Проверьте автопубликацию (2 минуты)

1. После push перейдите в репозиторий на GitHub
2. Откройте вкладку **Actions**
3. Вы увидите workflow **"Publish NuGet Packages"** в процессе выполнения
4. Дождитесь зеленой галочки ✅

**Время выполнения:** ~2-3 минуты

---

### Шаг 4: Проверьте опубликованные пакеты (1 минута)

1. В репозитории откройте **Packages** (справа на главной странице)
   
   **ИЛИ** перейдите напрямую:
   - https://github.com/orgs/Nix/packages
   - https://github.com/YOUR_USERNAME?tab=packages (если личный аккаунт)

2. Вы должны увидеть 4 пакета:
   - ✅ `FitCourse.Nix.BuildingBlocks` v1.0.X
   - ✅ `FitCourse.Nix.Messaging` v1.0.X
   - ✅ `FitCourse.Nix.Persistence` v1.0.X
   - ✅ `FitCourse.Nix.Contracts` v1.0.X

---

### Шаг 5: Настройте локальный доступ к пакетам (5 минут)

#### 5.1 Создайте Personal Access Token (PAT)

1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. **Generate new token (classic)**
3. Настройки:
   - **Note**: `FitCourse NuGet Packages`
   - **Expiration**: `90 days` (или больше)
   - **Scopes**: 
     - ✅ `read:packages`
     - ✅ `write:packages` (если планируете публиковать локально)
4. **Generate token**
5. **СОХРАНИТЕ ТОКЕН** — он больше не отобразится!

#### 5.2 Добавьте источник NuGet

**Windows PowerShell:**
```powershell
# Замените YOUR_USERNAME и YOUR_PAT на ваши данные!
dotnet nuget add source "https://nuget.pkg.github.com/nix-fit/index.json" `
  --name github-nix `
  --username YOUR_USERNAME `
  --password YOUR_PAT `
  --store-password-in-clear-text
```

**Linux / macOS:**
```bash
# Замените YOUR_USERNAME и YOUR_PAT на ваши данные!
dotnet nuget add source "https://nuget.pkg.github.com/nix-fit/index.json" \
  --name github-nix \
  --username YOUR_USERNAME \
  --password YOUR_PAT \
  --store-password-in-clear-text
```

---

### Шаг 6: Протестируйте установку пакетов (3 минуты)

```powershell
# Создайте тестовый проект
cd C:\Mine
mkdir test-buildingblocks
cd test-buildingblocks

dotnet new console -n TestNuGet
cd TestNuGet

# Установите пакеты
dotnet add package FitCourse.Nix.BuildingBlocks
dotnet add package FitCourse.Nix.Messaging
dotnet add package FitCourse.Nix.Persistence
dotnet add package FitCourse.Nix.Contracts

# Если все прошло успешно — пакеты установлены! ✅
dotnet restore
dotnet build

# Удалите тестовый проект
cd C:\Mine
Remove-Item -Recurse -Force test-buildingblocks
```

Если увидели ошибку:
- ❌ **401 Unauthorized** → проверьте PAT токен
- ❌ **404 Not Found** → проверьте URL источника (должен быть `/Nix/index.json`)
- ❌ **Package not found** → подождите 1-2 минуты, пакеты индексируются

---

## 🎉 Готово!

BuildingBlocks успешно опубликованы и готовы к использованию!

### Следующие шаги:

1. **Обновите текущие сервисы** в монорепо `C:\Mine\FitCourse`:
   - Замените `ProjectReference` на `PackageReference` в `.csproj`
   - Например:
     ```xml
     <!-- БЫЛО -->
     <ProjectReference Include="..\..\BuildingBlocks\Nix.BuildingBlocks\Nix.BuildingBlocks.csproj" />
     
     <!-- СТАЛО -->
     <PackageReference Include="FitCourse.Nix.BuildingBlocks" Version="1.0.*" />
     ```

2. **Создайте отдельные репозитории для сервисов**:
   - Начните с простого сервиса (AnalyticsService, CommentService)
   - Потом критичные (Gateway, UserService, CourseService)
   - Повторите процесс для всех сервисов

3. **Создайте Infrastructure репозиторий**:
   - `fitcourse-infrastructure`
   - Перенесите туда `docker-compose.yml`, Keycloak конфигурацию

---

## 📞 Помощь

### Частые проблемы

**Q: GitHub Actions не запускается**
A: Убедитесь что в Settings → Actions → General включен "Allow all actions"

**Q: Пакеты не видны в Packages**
A: Packages могут быть приватными по умолчанию. Перейдите в пакет → Package settings → Change visibility

**Q: dotnet restore не находит пакеты**
A: 
1. Проверьте источник: `dotnet nuget list source`
2. Проверьте PAT токен
3. Проверьте переменные окружения GITHUB_USERNAME, GITHUB_TOKEN

**Q: Как обновить версию пакетов?**
A: Просто сделайте коммит в `main` branch — версия обновится автоматически (1.0.1, 1.0.2...)

---

## 📚 Дополнительные ресурсы

- [GitHub Packages Documentation](https://docs.github.com/en/packages)
- [NuGet Package Versioning](https://learn.microsoft.com/en-us/nuget/concepts/package-versioning)
- [.NET Central Package Management](https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management)

---

**Удачи с миграцией! 🚀**

Если возникнут вопросы — проверьте README.md в репозитории или создайте Issue.

