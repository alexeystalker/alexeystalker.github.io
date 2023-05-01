---
share: true
tags:
 - NET/EFCore
---
# Создание записи
Необходимо дать пользователю создавать рецепты в приложении. Для этих целей будет служить форма, реализованая при помощи [[ch-8-creating-forms-using-tag-helpers|тег-хелперов]]. Ее содержимое отправляется на страницу Create.cshtml, на которой происходит [[ch-6-validating-on-server-for-safety|валидация]] переданных данных, и, при успешной валидации, вызывается сервис `RecipeService` для создания нового объекта `Recipe` в БД, последовательность действий изображена на рисунке.
![[Pasted image 20220426192148.png]]
Создание сущности в EF Core включает в себя добавление новой строки в отображаемую таблицу. Также будут корректно добавлены связанные сущности `Ingredients`.
Вот код метода:
```csharp
readonly AppDbContext _context; //внедряется в конструктор
public async Task<int> CreateRecipe(CreateRecipeCommand cmd)
{
	var recipe = new Recipe
	{
		Name = cmd.Name,
		TimeToCook = new TimeSpan(cmd.TimeToCookHrs, cmd.TimeToCookMins, 0),
		Method = cmd.Method,
		IsVegetarian = cmd.IsVegetarian,
		IsVegan = cmd.IsVegan,
		Ingredients = cmd.Ingredients?.Select(i =>
			new Ingredient
			{
				Name = i.Name,
				Quantity = i.Quantity,
				Unit = i.Unit
			}).ToList()
	};
	_context.Add(recipe); //Сообщаем EF Core, что нужно отслеживать новые сущности
	await _context.SaveChangesAsync(); //Запишем в БД
	return recipe.RecipeId; //поле RecipeId заполняется EF Core при сохранении
}
```
Здесь выполняются три шага:
1. создаем сущности `Recipe` и `Ingredient`;
2. добавляем их в список отслеживаемых сущностей EF Core;
3. вызываем `_context.SaveChangesAsync()`, чтобы выполнить команду `INSERT` и добавить строки в таблицы `Recipe` и `Ingredient`.

Если при взаимодействии с БД возникает проблема — например, не запущены миграции и БД не обновлена, — будет выброшено исключение. Это нужно учитывать в реальном коде.
Если же всё прошло удачно,  EF Core обновит все автоматически сгенерированные идентификаторы (`RecipeId` для `Recipe`, `IngredientId` для `Ingredient`). Можно вернуть идентификатор головной сущности на страницу Razor.