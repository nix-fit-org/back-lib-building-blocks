# 📋 План миграции FitCourse на отдельные репозитории

Этот документ описывает полный план разделения монорепозитория на независимые микросервисы.

---

## 🎯 Цели миграции

1. ✅ **Независимость команд** — каждая команда работает в своем репо
2. ✅ **Изолированный CI/CD** — деплой сервиса не затрагивает другие
3. ✅ **Версионирование** — каждый сервис имеет свою версию
4. ✅ **Уменьшение размера** — клонирование только нужного сервиса

---

## 📊 Текущее состояние

### Монорепозиторий `C:\Mine\FitCourse\`

**14 микросервисов:**
- Gateway
- UserService
- CourseService
- EnrollmentService
- MediaService
- NotificationService
- ProgressService
- HomeworkService
- CommentService
- ChatService
- AnalyticsService
- PurchaseService
- ReferralService
- AuthService

**3 общие библиотеки (BuildingBlocks):**
- ✅ Nix.BuildingBlocks → `FitCourse.Nix.BuildingBlocks` (NuGet)
- ✅ Nix.Messaging → `FitCourse.Nix.Messaging` (NuGet)
- ✅ Nix.Persistence → `FitCourse.Nix.Persistence` (NuGet)
- ✅ Nix.Contracts → `FitCourse.Nix.Contracts` (NuGet)

---

## 🗂️ Целевая структура репозиториев

```
GitHub Organization: Nix
│
├── back-lib-building-blocks           ✅ ГОТОВО
│   └── src/
│       ├── Nix.BuildingBlocks/
│       ├── Nix.Messaging/
│       ├── Nix.Persistence/
│       └── Nix.Contracts/
│
├── fitcourse-gateway                  ⏳ TODO
├── fitcourse-user-service             ⏳ TODO
├── fitcourse-course-service           ⏳ TODO
├── fitcourse-enrollment-service       ⏳ TODO
├── fitcourse-media-service            ⏳ TODO
├── fitcourse-notification-service     ⏳ TODO
├── fitcourse-progress-service         ⏳ TODO
├── fitcourse-homework-service         ⏳ TODO
├── fitcourse-comment-service          ⏳ TODO
├── fitcourse-chat-service             ⏳ TODO
├── fitcourse-analytics-service        ⏳ TODO
├── fitcourse-purchase-service         ⏳ TODO
├── fitcourse-referral-service         ⏳ TODO
│
└── fitcourse-infrastructure           ⏳ TODO
    ├── docker-compose.yml
    ├── docker-compose.dev.yml
    ├── docker-compose.prod.yml
    ├── keycloak/
    ├── scripts/
    └── k8s/
```

---

## 🚀 Фазы миграции

### ✅ Фаза 1: BuildingBlocks (ЗАВЕРШЕНА)

**Статус:** ✅ Готово  
**Время:** ~2 часа  
**Результат:** 
- Репозиторий `back-lib-building-blocks` создан
- 4 NuGet пакета опубликованы на GitHub Packages
- Документация и CI/CD настроены

**Пакеты:**
- `FitCourse.Nix.BuildingBlocks` v1.0.X
- `FitCourse.Nix.Messaging` v1.0.X
- `FitCourse.Nix.Persistence` v1.0.X
- `FitCourse.Nix.Contracts` v1.0.X

---

### ⏳ Фаза 2: Первый тестовый сервис (30-60 минут)

**Рекомендация:** Начните с **CommentService** или **AnalyticsService** (самые простые).

**Почему:** Низкий риск, мало зависимостей, быстрая проверка процесса.

**Шаги:**
1. Создайте репозиторий `fitcourse-comment-service` на GitHub
2. Скопируйте код сервиса из `src/Services/CommentService/`
3. Создайте `nuget.config` с источником GitHub Packages
4. Замените `ProjectReference` на `PackageReference`:
   ```xml
   <!-- БЫЛО -->
   <ProjectReference Include="..\..\BuildingBlocks\Nix.BuildingBlocks\Nix.BuildingBlocks.csproj" />
   
   <!-- СТАЛО -->
   <PackageReference Include="FitCourse.Nix.BuildingBlocks" Version="1.0.*" />
   ```
5. Создайте `.github/workflows/build-and-deploy.yml`
6. Создайте `Dockerfile` (если нужно обновить пути)
7. Создайте `README.md`
8. Push в GitHub
9. Проверьте CI/CD
10. Запустите локально: `dotnet restore && dotnet build && dotnet run`

**Критерии успеха:**
- ✅ Сервис собирается локально
- ✅ CI/CD проходит успешно
- ✅ Docker образ собирается
- ✅ Сервис запускается в docker-compose

---

### ⏳ Фаза 3: Критичные сервисы (по одному, ~1 час на сервис)

**Порядок миграции (по приоритету):**

1. **Gateway** (самый критичный, точка входа)
2. **UserService** (аутентификация/авторизация)
3. **CourseService** (основная бизнес-логика)
4. **EnrollmentService** (управление доступом)
5. **MediaService** (управление контентом)

**Для каждого сервиса:**
- Создайте отдельный репозиторий
- Следуйте процессу из Фазы 2
- Обновите docker-compose.yml в infrastructure репо

**⚠️ ВАЖНО:** 
- Миграция **по одному сервису**
- Проверка работоспособности после каждого
- Старый монорепо остается рабочим на время миграции

---

### ⏳ Фаза 4: Остальные сервисы (параллельно, ~3-4 часа)

После успешной миграции критичных сервисов, можно мигрировать остальные параллельно:

- NotificationService
- ProgressService
- HomeworkService
- ChatService
- AnalyticsService
- PurchaseService
- ReferralService

**Можно делегировать** членам команды, используя уже отработанный процесс.

---

### ⏳ Фаза 5: Infrastructure репозиторий (1 час)

**Цель:** Централизованное управление инфраструктурой.

**Создайте репозиторий `fitcourse-infrastructure`:**

```
fitcourse-infrastructure/
├── README.md
├── .env.example
├── docker-compose.yml              # Полная композиция всех сервисов
├── docker-compose.dev.yml          # Для разработки (с volume mounts)
├── docker-compose.prod.yml         # Для продакшена
├── keycloak/
│   └── realm-export.json
├── postgres/
│   └── init-databases.sql
├── scripts/
│   ├── start-all.sh
│   ├── stop-all.sh
│   ├── backup-db.sh
│   └── restore-db.sh
├── k8s/                            # Kubernetes манифесты (опционально)
│   ├── namespace.yaml
│   ├── gateway/
│   ├── user-service/
│   └── ...
└── monitoring/                     # Prometheus, Grafana (опционально)
    ├── prometheus.yml
    └── grafana/
```

**docker-compose.yml должен ссылаться на Docker образы из GitHub Container Registry:**

```yaml
services:
  gateway:
    image: ghcr.io/nix/fitcourse-gateway:latest
    # или
    # image: ghcr.io/nix/fitcourse-gateway:v1.2.3
    
  user-service:
    image: ghcr.io/nix/fitcourse-user-service:latest
    
  # ... и т.д.
```

---

## 📦 Шаблон .csproj для сервисов

Каждый сервис должен использовать PackageReference вместо ProjectReference:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <!-- BuildingBlocks через NuGet -->
    <PackageReference Include="FitCourse.Nix.BuildingBlocks" Version="1.0.*" />
    <PackageReference Include="FitCourse.Nix.Messaging" Version="1.0.*" />
    <PackageReference Include="FitCourse.Nix.Persistence" Version="1.0.*" />
    <PackageReference Include="FitCourse.Nix.Contracts" Version="1.0.*" />
    
    <!-- Остальные пакеты -->
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" />
    <!-- ... -->
  </ItemGroup>

</Project>
```

---

## 📦 Шаблон nuget.config для сервисов

Каждый сервис должен иметь `nuget.config` в корне:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="github-nix" value="https://nuget.pkg.github.com/nix-fit/index.json" protocolVersion="3" />
  </packageSources>
  <packageSourceCredentials>
    <github-nix>
      <add key="Username" value="%GITHUB_USERNAME%" />
      <add key="ClearTextPassword" value="%GITHUB_TOKEN%" />
    </github-nix>
  </packageSourceCredentials>
</configuration>
```

---

## 🔄 Шаблон GitHub Actions для сервисов

`.github/workflows/build-and-deploy.yml`:

```yaml
name: Build and Deploy

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  DOTNET_VERSION: '9.0.x'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      packages: read
      contents: read
      
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}
          
      - name: Setup NuGet Source
        run: |
          dotnet nuget add source "https://nuget.pkg.github.com/nix-fit/index.json" \
            --name github-nix \
            --username ${{ github.actor }} \
            --password ${{ secrets.GITHUB_TOKEN }} \
            --store-password-in-clear-text
          
      - name: Restore
        run: dotnet restore
        
      - name: Build
        run: dotnet build -c Release --no-restore
        
      - name: Test
        run: dotnet test -c Release --no-build
        
      - name: Publish
        run: dotnet publish -c Release --no-build -o ./publish
        
  docker:
    needs: build
    runs-on: ubuntu-latest
    permissions:
      packages: write
      contents: read
      
    steps:
      - uses: actions/checkout@v4
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

---

## ✅ Checklist миграции

### BuildingBlocks
- [x] Репозиторий создан
- [x] NuGet пакеты опубликованы
- [x] CI/CD настроен
- [x] Документация готова
- [x] Локальный доступ настроен

### Каждый сервис
- [ ] Репозиторий создан на GitHub
- [ ] Код скопирован из монорепо
- [ ] `nuget.config` создан
- [ ] `ProjectReference` заменены на `PackageReference`
- [ ] `.github/workflows/` настроен
- [ ] `Dockerfile` обновлен
- [ ] `README.md` создан
- [ ] Локально собирается (`dotnet build`)
- [ ] CI/CD проходит успешно
- [ ] Docker образ публикуется в GHCR
- [ ] Обновлен `docker-compose.yml` в infrastructure

### Infrastructure
- [ ] Репозиторий создан
- [ ] `docker-compose.yml` для всех сервисов
- [ ] Keycloak конфигурация перенесена
- [ ] Скрипты управления созданы
- [ ] Документация по запуску всей системы

---

## 📞 Помощь и поддержка

### Проблемы при миграции

**Q: Сервис не находит BuildingBlocks пакеты**
A: Проверьте:
1. `nuget.config` существует в корне проекта
2. Переменные `GITHUB_USERNAME` и `GITHUB_TOKEN` установлены
3. PAT токен имеет scope `read:packages`

**Q: CI/CD падает с ошибкой 401**
A: GitHub Actions должен использовать `${{ secrets.GITHUB_TOKEN }}`, а не PAT

**Q: Docker build не находит пакеты**
A: В `Dockerfile` добавьте `RUN` шаг для настройки NuGet source перед `dotnet restore`

**Q: Как обновить версию BuildingBlocks в сервисе?**
A: 
1. Используйте `Version="1.0.*"` для автообновления патч-версий
2. Или явно укажите версию: `Version="1.0.5"`
3. Выполните `dotnet restore --force-evaluate`

---

## 🎯 Целевое состояние (после миграции)

```
✅ 15+ независимых репозиториев
✅ Каждый с собственным CI/CD
✅ BuildingBlocks как NuGet пакеты
✅ Centralized infrastructure управление
✅ Версионирование каждого сервиса
✅ Изолированные деплойменты
```

---

**Время полной миграции:** ~20-30 часов (включая тестирование)

**Параллелизация:** После Фазы 3 можно распределить работу между несколькими разработчиками.

---

**Удачи с миграцией! 🚀**

