---
share: true
tags:
 - NET/EFCore
---
# Выбор провайдера базы данных и установка EF Core
EF Core поддерживает ряд баз данных с помощью модели провайдера.

Добавление поддержки БД включает в себя добавление правильного пакета NuGet в проект. Например[^1]:
- *PostgreSQL* — Npgsql.EntityFrameworkCore.PostgreSQL;
- *Microsoft SQL Server* — Microsoft.EntityFrameworkCore.SqlServer;
- *MySQL* — MySql.Data.EntityFrameworkCore;
- *SQLite* — Microsoft.EntityFrameworkCore.SQLite.

[^1]:Полный список известных Microsoft провайдеров [здесь](https://docs.microsoft.com/en-us/ef/core/providers/?tabs=dotnet-core-cli)

Так как EF Core является модульной библиотекой, то для полноценной работы нужно установить несколько пакетов. Например, для нашего [[ch-12-sample-application|приложения]] необходимо два пакета:
- *Microsoft.EntityFrameworkCore.SqlServer* — основной пакет провайдера. Также содержит ссылку на базовый пакет EF Core;
- *Microsoft.EntityFrameworkCore.Design* — содержит компоненты времени проектирования EF Core.