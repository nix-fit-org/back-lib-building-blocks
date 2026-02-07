# Changelog

Все значимые изменения в BuildingBlocks пакетах будут документироваться в этом файле.

Формат основан на [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
и проект придерживается [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0] - 2025-10-30

### Added
- ✨ **Nix.BuildingBlocks** - Базовые DDD building blocks
  - `AggregateRoot<TId>` - базовый класс для агрегатов
  - `BaseEntity<TId>` - базовый класс для сущностей
  - `DomainEvent` - базовый класс для доменных событий
  - `ValueObject` - базовый класс для value objects
  - `Money` - value object для денежных сумм
  - `Result<T>` - railway-oriented programming паттерн
  - `Guard` - guard clauses для валидации
  - Outbox паттерн для гарантированной доставки событий
    - `OutboxEvent` - модель события в Outbox
    - `IOutboxRepository` - интерфейс репозитория
    - `IOutboxProcessor` - интерфейс обработчика
    - `IOutboxEventSerializer` - интерфейс сериализатора
    - `JsonOutboxEventSerializer` - JSON реализация
  - `BaseException` - базовый класс исключений

- ✨ **Nix.Messaging** - Messaging abstractions
  - `IEventBus` - абстракция шины событий
  - `MassTransitEventBus` - реализация через MassTransit + RabbitMQ
  - `ServiceCollectionExtensions` - расширения для быстрой настройки

- ✨ **Nix.Persistence** - Persistence patterns
  - `IRepository<TEntity>` - generic repository интерфейс
  - `Repository<TEntity>` - базовая реализация с EF Core
  - `IUnitOfWork` - unit of work интерфейс
  - `GenericUnitOfWork` - реализация UoW с EF Core
  - `ServiceCollectionExtensions` - расширения для DI

- ✨ **Nix.Contracts** - Integration Events
  - Access Events (AccessGrantedEvent, AccessRevokedEvent)
  - Course Events (CoursePublished, CourseUpdated, ModuleAdded, etc.)
  - Content Events (ContentScheduled, ContentPublished, ContentArchived)
  - Progress Events (LessonViewed, LessonCompleted)
  - Order Events (OrderCreated, OrderCompleted, OrderRefunded)
  - DTOs для межсервисного взаимодействия

### Infrastructure
- 🚀 GitHub Actions workflow для автопубликации в GitHub Packages
- 📦 Автоматическое версионирование (main → 1.0.X, tags → X.Y.Z)
- 📚 Полная документация (README, GETTING_STARTED, MIGRATION_PLAN)
- 🔒 Поддержка приватных NuGet пакетов через GitHub Packages

---

## Типы изменений

- `Added` - новая функциональность
- `Changed` - изменения в существующей функциональности
- `Deprecated` - функциональность, которая будет удалена в будущих версиях
- `Removed` - удаленная функциональность
- `Fixed` - исправление багов
- `Security` - исправления уязвимостей безопасности

---

[Unreleased]: https://github.com/nix-fit-org/back-lib-building-blocks/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/nix-fit-org/back-lib-building-blocks/releases/tag/v1.0.0

