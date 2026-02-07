# ⚡ Быстрый старт за 10 минут

## Шаг 1: Создайте репозиторий на GitHub (2 мин)

1. Перейдите: https://github.com/new
2. Repository name: `back-lib-building-blocks`
3. Private
4. **НЕ ДОБАВЛЯЙТЕ** README/gitignore/license
5. Create repository
6. Скопируйте URL: `https://github.com/nix-fit-org/back-lib-building-blocks.git`

## Шаг 2: Push кода (2 мин)

```powershell
cd C:\Mine\back-lib-building-blocks

git init
git add .
git commit -m "Initial commit: BuildingBlocks packages"
git remote add origin https://github.com/nix-fit-org/back-lib-building-blocks.git
git branch -M main
git push -u origin main
```

⚠️ **ЗАМЕНИТЕ** URL на ваш!

## Шаг 3: Дождитесь публикации (3 мин)

1. GitHub → Actions → дождитесь ✅
2. GitHub → Packages → увидите 4 пакета

## Шаг 4: Настройте локальный доступ (3 мин)

### Создайте PAT токен:
GitHub → Settings → Developer settings → Personal access tokens → Generate new token (classic)
- Scopes: `read:packages`
- Сохраните токен!

### Добавьте источник:

```powershell
# ЗАМЕНИТЕ YOUR_USERNAME и YOUR_PAT!
dotnet nuget add source "https://nuget.pkg.github.com/nix-fit/index.json" `
  --name github-nix `
  --username YOUR_USERNAME `
  --password YOUR_PAT `
  --store-password-in-clear-text
```

## Шаг 5: Проверьте работу

```powershell
dotnet new console -n Test
cd Test
dotnet add package FitCourse.Nix.BuildingBlocks
dotnet restore
```

Если увидели успешное восстановление — **готово! ✅**

---

## 🎯 Дальше

Читайте:
- `GETTING_STARTED.md` — подробные инструкции
- `MIGRATION_PLAN.md` — план миграции сервисов
- `README.md` — документация пакетов

**Следующий шаг:** Миграция первого сервиса (см. MIGRATION_PLAN.md → Фаза 2)

