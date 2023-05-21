---
share: true
tags:
 - NET/EFCore
---
# Создание модели данных
Некоторые инструменты ORM требуют, чтобы сущности были унаследованы от определенного базового класса, или же декорированы соответствующими атрибутами. EF Core в значительной степени отдает предпочтение подходу с использованием соглашений.
Рассмотрим пример с сущностями нашего [[ch-12-sample-application|приложения]]:
```csharp
public class Recipe
{
	public int RecipeId { get; set; }
	public string Name { get; set; }
	public TimeSpan TimeToCook { get; set; }
	public bool IsDeleted { get; set; }
	public string Method { get; set; }
	public ICollection<Ingredient> Ingredients { get; set; }
}
public class Ingredient
{
	public int IngredientId { get; set; }
	public int RecipeId { get; set; }
	public string Name { get; set; }
	public decimal Quantity { get; set; }
	public string Unit { get; set; }
}
```
Эти классы соответствуют определенным соглашениям[^1], которые EF Core использует для создания картины отображаемой БД. Например, у класса `Recipe` есть свойство `RecipeId`, а у `Ingredient` — `IngredientId`. Это — шаблон указания на *первичный ключ* таблицы.
Свойство `RecipeId` в классе `Ingredient` интерпретируется как *внешний ключ*, указывающий на `Recipe`. В сочетании с `ICollection<Ingredient>` в классе `Recipe` это указывает на связь "многие к одному".
![[Pasted image 20220417143042.png]]

[^1]: Полную документацию по соглашениям см. [здесь](https://docs.microsoft.com/en-us/ef/core/modeling/)

Также можно использовать атрибуты `DataAnnotations` для декорирования классов сущностей для управления процессом отображения.

Помимо сущностей, также необходимо определить `DbContext`. Создадим класс `AppDbContext`, унаследуем его от `DbContext` и определим набор свойств типа `DbSet<>` для каждой из сущностей верхнего уровня:
```csharp
public class AppDbContext : DbContext
{
	public AppDbContext(DbContextOptions<AppDbContext> options) 
		: base(options) {}
	public DbSet<Recipe> Recipes { get; set; }
}
```

Как можно видеть, класс `Ingredient` не указан в `AppDbContext`, но он будет смоделирован, так как он есть в `Recipe`. Доступ к сущностям `Ingredient` будет осуществляться только через свойство `Ingredients` сущности `Recipe`.

Это типичный Code-first подход. Однако, если действующая БД уже есть, можно по ней сгенерировать сущности EF и класс `DbContext`. [Подробнее](https://docs.microsoft.com/en-us/ef/core/managing-schemas/scaffolding?tabs=dotnet-core-cli)