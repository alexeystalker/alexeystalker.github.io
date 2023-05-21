---
share: true
tags:
 - NET/ASPNETCore/Identity
---
# Обновление модели данных EF Core для поддержки Identity
Прежде всего, создадим класс `ApplicationUser` — наследника `IdentityUser`:
```csharp
public class ApplicationUser : IdentityUser { }
```
[[ch-14-asp-net-core-identity-data-model|Ранее мы видели]], что Identity предоставляет класс `IdentityDbContext`с необходимыми `DbSet<T>`. Поэтому обновим `DbContext` приложения, унаследовав его от `IdentityDbContext`:
```csharp
public class AppDbContext : IdentityDbContext<ApplicationUser>
{
	public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) {}
	public DbSet<Recipe> Recipes { get; set; }
}
```
Фактически, мы обновили класс контекста, добавив загрузку новых сущностей в модель данных EF Core. [[ch-12-adding-a-second-migration|Как было показано ранее]], после изменения модели данных необходимо создать новую миграцию и применить к базе данных.
```bash
dotnet ef migrations add AddIdentitySchema
```
