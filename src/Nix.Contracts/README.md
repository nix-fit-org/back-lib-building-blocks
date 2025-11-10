# Nix.Contracts

Shared contracts для integration events между микросервисами FitCourse.

## 📦 Установка

```bash
dotnet add package FitCourse.Nix.Contracts
```

## 🎯 Назначение

Этот пакет содержит **только общие абстракции** для межсервисного взаимодействия через integration events. Он предоставляет единую структуру для всех событий, которыми обмениваются микросервисы.

## ✅ Что ДОЛЖНО быть в этом пакете

- ✅ Базовые интерфейсы для integration events (`IIntegrationEvent`)
- ✅ Базовые классы для events (`IntegrationEventBase`)
- ✅ Общие value objects, используемые **≥3 сервисами**
- ✅ Shared константы для межсервисного взаимодействия (`ResourceTypes`)
- ✅ Общие enum'ы для integration events (статусы, типы)

## ❌ Что НЕ ДОЛЖНО быть в этом пакете

- ❌ Domain-специфичные enums конкретных сервисов
- ❌ DTOs с бизнес-логикой конкретного bounded context
- ❌ Типы с namespace `ServiceName.Domain.*`
- ❌ Aggregate-специфичные события
- ❌ Логика обработки событий (handlers)

**Золотое правило:** Если тип используется только в одном сервисе или его bounded context, он НЕ должен быть здесь.

## 🚀 Использование

### 1. Базовый интерфейс IIntegrationEvent

```csharp
using Nix.Contracts.Events;

public interface IIntegrationEvent
{
    Guid EventId { get; }          // Уникальный идентификатор события
    DateTime OccurredAt { get; }   // Время возникновения (UTC)
    string EventType { get; }      // Тип события: "service.entity.action.version"
}
```

### 2. Создание собственного integration event

```csharp
using Nix.Contracts.Events;

namespace EnrollmentService.Contracts;

/// <summary>
/// Событие о предоставлении доступа к курсу
/// </summary>
public sealed record AccessGrantedEvent : IntegrationEventBase
{
    /// <summary>
    /// Тип события в формате: service.entity.action.version
    /// </summary>
    public override string EventType => "enrollment.access.granted.v1";
    
    /// <summary>
    /// Идентификатор пользователя
    /// </summary>
    public required Guid UserId { get; init; }
    
    /// <summary>
    /// Идентификатор курса
    /// </summary>
    public required Guid CourseId { get; init; }
    
    /// <summary>
    /// Дата начала доступа (UTC)
    /// </summary>
    public required DateTime AccessStartDate { get; init; }
    
    /// <summary>
    /// Дата окончания доступа (UTC), null = бессрочный
    /// </summary>
    public DateTime? AccessEndDate { get; init; }
}
```

### 3. Использование в сервисе

```csharp
// В EnrollmentService
var @event = new AccessGrantedEvent
{
    UserId = enrollment.UserId,
    CourseId = enrollment.CourseId,
    AccessStartDate = DateTime.UtcNow,
    AccessEndDate = DateTime.UtcNow.AddMonths(6)
};

await _eventBus.PublishAsync(@event);
```

```csharp
// В CourseService (consumer)
public class AccessGrantedEventHandler : IConsumer<AccessGrantedEvent>
{
    public async Task Consume(ConsumeContext<AccessGrantedEvent> context)
    {
        var @event = context.Message;
        
        // Обработка события
        await _courseAccessService.GrantAccessAsync(
            @event.UserId, 
            @event.CourseId, 
            @event.AccessEndDate
        );
    }
}
```

### 4. Использование общих констант

```csharp
using Nix.Contracts.Common;

// Вместо magic strings
var resourceType = ResourceTypes.Course;
var lessonType = ResourceTypes.Lesson;
```

## 📋 Именование EventType

Используйте следующий формат:

```
service.entity.action.version
```

**Примеры:**
- `enrollment.access.granted.v1` — доступ предоставлен
- `enrollment.access.revoked.v1` — доступ отозван
- `course.lesson.completed.v1` — урок завершен
- `course.certificate.issued.v1` — сертификат выдан
- `user.profile.updated.v1` — профиль обновлен

**Преимущества версионирования:**
- Поддержка нескольких версий одновременно
- Плавная миграция между версиями
- Явная индикация breaking changes

## 🏗️ Архитектурные принципы

### Почему record?

Integration events являются **immutable** по своей природе:
- События описывают то, что **уже произошло**
- Они не должны изменяться после создания
- `record` в C# идеально подходит для immutable данных

### Почему abstract EventType?

```csharp
public abstract string EventType { get; }
```

Это заставляет каждое событие явно декларировать свой тип, что:
- Предотвращает ошибки с неуникальными типами
- Делает тип события частью контракта
- Облегчает routing в message brokers

### Автоматическая генерация метаданных

```csharp
public Guid EventId { get; init; } = Guid.NewGuid();
public DateTime OccurredAt { get; init; } = DateTime.UtcNow;
```

Каждое событие автоматически получает:
- Уникальный ID для трейсинга
- Timestamp для аудита и отладки

## 🔄 Миграция существующих событий

**Было (в EnrollmentService):**
```csharp
public sealed class AccessGrantedEvent : DomainEvent
{
    public Guid UserId { get; set; }
    public Guid CourseId { get; set; }
}
```

**Стало:**
```csharp
using Nix.Contracts.Events;

public sealed record AccessGrantedEvent : IntegrationEventBase
{
    public override string EventType => "enrollment.access.granted.v1";
    
    public required Guid UserId { get; init; }
    public required Guid CourseId { get; init; }
}
```

**Изменения:**
1. ✅ Наследуется от `IntegrationEventBase`
2. ✅ Использует `record` вместо `class`
3. ✅ Properties с `init` вместо `set`
4. ✅ Явный `EventType`
5. ✅ Убраны `[Required]` attributes (используем `required` keyword)

## 🔗 Связанные пакеты

- **FitCourse.Nix.BuildingBlocks** — Domain entities, Value Objects, Result
- **FitCourse.Nix.Messaging** — MassTransit integration, IEventBus
- **FitCourse.Nix.Persistence** — Repository pattern, Unit of Work

## 📚 Дополнительно

### Когда создавать новую версию события?

Создавайте `v2`, `v3` и т.д., когда:
- Меняется структура данных (breaking change)
- Добавляются обязательные поля
- Удаляются существующие поля

**Не требуется новая версия** при:
- Добавлении optional полей
- Изменении документации
- Исправлении багов в consumers

### Пример версионирования

```csharp
// v1
public sealed record CourseCreatedEvent : IntegrationEventBase
{
    public override string EventType => "course.created.v1";
    public required Guid CourseId { get; init; }
    public required string Title { get; init; }
}

// v2 - добавлены новые поля
public sealed record CourseCreatedEventV2 : IntegrationEventBase
{
    public override string EventType => "course.created.v2";
    public required Guid CourseId { get; init; }
    public required string Title { get; init; }
    public required string Description { get; init; }  // NEW
    public required decimal Price { get; init; }       // NEW
}
```

## 📞 Поддержка

Вопросы и предложения: https://github.com/nix-fit/building-blocks/issues

## 📄 Лицензия

MIT License

