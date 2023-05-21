---
share: true
tags:
 - NET/EFCore
---
# Загрузка списка записей
Напишем код для просмотра списка рецептов [[ch-12-sample-application|приложения]]. Создадим метод в `RecipeService`, который будет возвращать список рецептов в общем виде, состоящем из `RecipeId`, `Name` и `TimeToCook`.
Свойство `DbSet<Recipe>` в `AppDbContext` реализует `IQueryable`, значит, можно использовать обычные LINQ методы типа `Where()` и `Select()`. EF Core преобразует их в инструкцию SQL для осуществления запроса к БД в момент вызова функции *выполнения*, такой как `ToListAsync()`, `ToArrayAsync()` и т.п.
```csharp
public async Task<ICollection<RecipeSummaryViewModel>> GetRecipes()
{
	return await _context.Recipes
		.Where(r => !r.Is.Deleted)
		.Select(r => new RecipeSummaryViewModel
				{
					Id = r.RecipeId,
					Name = r.Name,
					TimeToCook = $"{r.TimeToCook.TotalMinutes} mins"
				})
		.ToListAsync();
}
```