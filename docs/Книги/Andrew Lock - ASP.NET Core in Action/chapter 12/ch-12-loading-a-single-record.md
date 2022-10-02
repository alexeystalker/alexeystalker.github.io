---
share: true
tags:
 - NET/EFCore
---
# Загрузка одной записи
Напишем код, извлекающий детали рецепта по его идентификатору. Для этого нам нужно использовать LINQ-выражение `Where` для ограничения запроса одним рецептом, затем преобразовать полученный объект к модели отображения.
```csharp
public async Task<RecipeDetailViewModel> GetRecipeDetail(int id)
{
	return await _context.Recipes
		.Where(x => x.RecipeId == id)
		.Select(x => new RecipeDetailViewModel
				{
					Id = x.RecipeId,
					Name = x.Name,
					Method = x.Method,
					Ingredients = x.Ingredients
						.Select(item => new RecipeDetailViewModel.Item
								{
									Name = item.Name,
									Quantity = $"{item.Quantity} {item.Unit}"
								})
				})
		.SingleOrDefaultAsync();
}
```
Здесь мы отображаем не только `Recipe` в `RecipeDetailViewModel`, но также и ингредиенты для рецепта.  При этом EF Core сам решает, как лучше создать SQL-запросы для получения данных.