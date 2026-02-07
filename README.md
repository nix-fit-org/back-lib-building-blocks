# FitCourse BuildingBlocks

Общие библиотеки и контракты для микросервисов платформы FitCourse.

## 📦 Пакеты

### FitCourse.Nix.BuildingBlocks
[![NuGet](https://img.shields.io/nuget/v/FitCourse.Nix.BuildingBlocks.svg)](https://github.com/nix-fit-org/buildingblocks/packages)

**Базовые DDD building blocks:**
- `AggregateRoot`, `BaseEntity` — базовые классы для сущностей
- `DomainEvent` — доменные события
- `ValueObject` — value objects (Money и базовый класс)
- `Result` — railway-oriented programming паттерн
- `Guard` — guard clauses для валидации
- Outbox паттерн для гарантированной доставки событий
- `BaseException` — базовый класс исключений

**Установка:**
```bash
dotnet add package FitCourse.Nix.BuildingBlocks
```

---

### FitCourse.Nix.Messaging  
[![NuGet](https://img.shields.io/nuget/v/FitCourse.Nix.Messaging.svg)](https://github.com/nix-fit-org/buildingblocks/packages)

**Messaging abstractions для асинхронного взаимодействия:**
- `IEventBus` — абстракция шины событий
- `MassTransitEventBus` — реализация через MassTransit + RabbitMQ
- Service collection extensions для быстрой настройки

**Установка:**
```bash
dotnet add package FitCourse.Nix.Messaging
```

**Пример использования:**
```csharp
// В Program.cs
services.AddMassTransitEventBus(configuration);

// В сервисе
public class MyService
{
    private readonly IEventBus _eventBus;
    
    public MyService(IEventBus eventBus)
    {
        _eventBus = eventBus;
    }
    
    public async Task PublishEvent(CancellationToken ct)
    {
        await _eventBus.Publish(new MyIntegrationEvent(), ct);
    }
}
```

---

### FitCourse.Nix.Persistence
[![NuGet](https://img.shields.io/nuget/v/FitCourse.Nix.Persistence.svg)](https://github.com/nix-fit-org/buildingblocks/packages)

**Persistence patterns для работы с базой данных:**
- `IRepository<T>` — generic repository
- `Repository<T>` — базовая реализация с EF Core
- `IUnitOfWork` — unit of work pattern
- `GenericUnitOfWork` — реализация UoW

**Установка:**
```bash
dotnet add package FitCourse.Nix.Persistence
```

**Пример использования:**
```csharp
// В Program.cs
services.AddPersistence<MyDbContext>();

// В сервисе
public class MyService
{
    private readonly IRepository<MyEntity> _repository;
    private readonly IUnitOfWork _unitOfWork;
    
    public async Task CreateEntity(MyEntity entity, CancellationToken ct)
    {
        await _repository.AddAsync(entity, ct);
        await _unitOfWork.SaveChangesAsync(ct);
    }
}
```

---

### FitCourse.Nix.Contracts
[![NuGet](https://img.shields.io/nuget/v/FitCourse.Nix.Contracts.svg)](https://github.com/nix-fit-org/buildingblocks/packages)

**Контракты интеграционных событий для межсервисного взаимодействия:**

**События доступа (Access Events):**
- `AccessGrantedEvent` — выдан доступ к ресурсу
- `AccessRevokedEvent` — отозван доступ к ресурсу

**События курсов (Course Events):**
- `CoursePublished` — курс опубликован
- `CourseUpdated` — курс обновлен
- `ModuleAdded` — модуль добавлен
- `LessonVersionPublished` — версия урока опубликована
- `TagAdded`, `TagRemoved` — управление тегами

**События контента (Content Events):**
- `ContentScheduled` — контент запланирован
- `ContentPublished` — контент опубликован
- `ContentArchived` — контент архивирован

**События прогресса (Progress Events):**
- `LessonViewed` — урок просмотрен
- `LessonCompleted` — урок завершен

**События заказов (Order Events):**
- `OrderCreated` — заказ создан
- `OrderCompleted` — заказ завершен
- `OrderRefunded` — заказ возвращен

**Установка:**
```bash
dotnet add package FitCourse.Nix.Contracts
```

---

## 🚀 Быстрый старт

### 1. Настройка аутентификации для GitHub Packages

#### Локальная разработка

Создайте Personal Access Token (PAT) на GitHub:
1. GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate new token
3. Scopes: `read:packages`, `write:packages` (если будете публиковать)
4. Сохраните токен!

Добавьте источник NuGet:

**Windows (PowerShell):**
```powershell
dotnet nuget add source "https://nuget.pkg.github.com/nix-fit/index.json" `
  --name github-nix `
  --username YOUR_GITHUB_USERNAME `
  --password YOUR_GITHUB_PAT `
  --store-password-in-clear-text
```

**Linux / macOS:**
```bash
dotnet nuget add source "https://nuget.pkg.github.com/nix-fit/index.json" \
  --name github-nix \
  --username YOUR_GITHUB_USERNAME \
  --password YOUR_GITHUB_PAT \
  --store-password-in-clear-text
```

#### CI/CD (GitHub Actions)

В GitHub Actions используйте `${{ secrets.GITHUB_TOKEN }}` — он доступен автоматически:

```yaml
- name: Setup NuGet
  run: |
    dotnet nuget add source "https://nuget.pkg.github.com/nix-fit/index.json" \
      --name github-nix \
      --username ${{ github.actor }} \
      --password ${{ secrets.GITHUB_TOKEN }} \
      --store-password-in-clear-text
```

### 2. Установка пакетов

```bash
dotnet add package FitCourse.Nix.BuildingBlocks
dotnet add package FitCourse.Nix.Messaging
dotnet add package FitCourse.Nix.Persistence
dotnet add package FitCourse.Nix.Contracts
```

### 3. Использование в проекте

Создайте `nuget.config` в корне вашего решения:

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

Установите переменные окружения:
```bash
# Windows PowerShell
$env:GITHUB_USERNAME = "your-username"
$env:GITHUB_TOKEN = "your-pat-token"

# Linux / macOS
export GITHUB_USERNAME="your-username"
export GITHUB_TOKEN="your-pat-token"
```

Теперь просто:
```bash
dotnet restore
dotnet build
```

---

## 🔄 Версионирование

Проект использует автоматическое версионирование:

- **`main` branch** → стабильные версии `1.0.X`
- **`develop` branch** → preview версии `1.0.X-preview`
- **git tags `vX.Y.Z`** → релизные версии `X.Y.Z`

### Создание нового релиза

```bash
git tag v1.1.0
git push origin v1.1.0
```

GitHub Actions автоматически опубликует версию `1.1.0`.

---

## 🛠️ Разработка

### Локальная сборка и тестирование

```bash
# Клонировать репозиторий
git clone https://github.com/nix-fit-org/back-lib-building-blocks.git
cd back-lib-building-blocks

# Восстановить зависимости
dotnet restore

# Собрать все проекты
dotnet build

# Упаковать в NuGet пакеты
dotnet pack -c Release -o ./packages

# Установить локально для тестирования
dotnet nuget add source ./packages --name local
dotnet add package FitCourse.Nix.BuildingBlocks --source local
```

### Структура проекта

```
back-lib-building-blocks/
├── .github/
│   └── workflows/
│       └── publish.yml          # CI/CD для автопубликации
├── src/
│   ├── Nix.BuildingBlocks/      # DDD building blocks
│   ├── Nix.Messaging/           # Event bus abstractions
│   ├── Nix.Persistence/         # Repository & UoW patterns
│   └── Nix.Contracts/           # Integration events
├── Directory.Packages.props     # Централизованное управление версиями
├── .gitignore
└── README.md
```

---

## 📚 Архитектурные принципы

### DDD (Domain-Driven Design)
Все building blocks следуют принципам DDD:
- Aggregate Root для управления жизненным циклом сущностей
- Domain Events для реакции на изменения в домене
- Value Objects для бизнес-концептов

### CQRS
Поддержка Command Query Responsibility Segregation через MediatR.

### Event Sourcing
Outbox паттерн для гарантированной доставки событий.

### Transactional Outbox Pattern
Гарантия атомарности: доменные изменения + публикация событий в одной транзакции.

---

## 🤝 Contributing

1. Fork репозитория
2. Создайте feature branch: `git checkout -b feature/my-feature`
3. Commit изменения: `git commit -am 'Add new feature'`
4. Push в branch: `git push origin feature/my-feature`
5. Создайте Pull Request

---

## 📝 Changelog

### v1.0.0 (Initial Release)
- ✅ Nix.BuildingBlocks — базовые DDD паттерны
- ✅ Nix.Messaging — интеграция с MassTransit
- ✅ Nix.Persistence — Repository и UnitOfWork
- ✅ Nix.Contracts — интеграционные события

---

## 📞 Контакты

- GitHub: [Nix Organization](https://github.com/nix-fit)
- Issues: [Report Bug or Request Feature](https://github.com/nix-fit-org/buildingblocks/issues)

---

**Сделано с ❤️ командой Nix для FitCourse**

