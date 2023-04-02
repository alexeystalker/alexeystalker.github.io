---
share: true
tags:
 - NET/EFCore
---
# Добавляем EF Core в приложение
Добавим EF Core в [[ch-12-sample-application|простое приложение]] Razor pages.

Процесс добавления EF Core в приложение состоит из нескольких этапов.
1. [[ch-12-choosing-db-provider-and-installing-ef-core|Выбрать провайдера БД]], например Postgres, SQLite или MS SQL Server.
2. [[ch-12-choosing-db-provider-and-installing-ef-core|Установить]] пакеты NuGet для EF Core.
3. [[ch-12-building-data-model|Спроектировать]] класс `DbContext` своего приложения и сущности, составляющие модель данных.
4. [[ch-12-registering-data-context|Зарегистрировать]] этот класс в [[di-container|контейнере зависимостей]] ASP.NET Core.
5. [[ch-12-creating-your-first-migration|Использовать]] EF Core для создания *миграции*, описывающей модель данных.
6. Применить миграцию к БД, чтобы обновить схему БД.
