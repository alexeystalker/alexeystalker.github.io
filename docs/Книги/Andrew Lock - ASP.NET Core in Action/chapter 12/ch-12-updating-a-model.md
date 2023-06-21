---
share: true
tags:
 - NET/EFCore
---
# Обновление модели
Обновление сущностей после их изменения — обычно самая сложная часть CRUD-операций, поскольку переменных очень много.
![[Pasted image 20220430135424.png]]
Опустим аспект обновления связей — решение этой проблемы зависит от конкретной модели данных. Рассмотрим обновление самой сущности `Recipe`.
В случае веб-приложений при обновлении сущности обычно выполняются три шага:
1. чтение сущности из БД;
2. изменение свойств сущности;
3. сохранение изменений в БД.

Инкапсулируем эти шаги в методе `UpdateRecipe`.
```csharp
public async Task UpdateRecipe(UpdateRecipeCommand cmd)
{
	var recipe = await _context.Recipes.FindAsync(cmd.Id);
	if (recipe == null)
	{
		throw new Exception("Unable to find the recipe");
	}
	UpdateRecipe(recipe, cmd);
	await _context.SaveChangesAsync();
}

private static void UpdateRecipe(Recipe recipe, UpdateRecipeCommand cmd)
{
	recipe.Name = cmd.Name;
	recipe.TimeToCook = new TimeSpan(cmd.TimeToCookHrs, cmd.TimeToCookMins, 0);
	recipe.Method = cmd.Method;
	recipe.IsVegetarian = cmd.IsVegetarian;
	recipe.IsVegan = cmd.IsVegan;
}
```
Как можно увидеть, здесь использован метод `FindAsync()` для поиска по идентификатору. Он аналогичен поиску через метод `Where`:
```csharp
_context.Recipes.Where(r => r.RecipeId == cmd.Id).FirstOrDefault();
```
Однако, кроме этого, метод `Find` предварительно проверяет не отслеживается ли эта сущность в `DbContext`, и если да, она возвращается немедленно, без дополнительного запроса к БД.
Также нужно упомянуть, что при вызове `SaveChangesAsync()` EF Core формирует SQL инструкцию UPDATE только из измененных свойств. Это происходит благодаря тому, что EF Core отслеживает состояние всех отслеживаемых сущностей.

Также стоит рассмотреть вопрос удаления рецептов. Самым распространенным паттерном здесь будет “мягкое” удаление — сущность на самом деле не удаляется из БД, а помечается удаленной, и при последующих запросах не возвращается.
```csharp
public async Task DeleteRecipe(int recipeId)
{
	var recipe = await _context.Recipes.FindAsync(recipeId);
	if (recipe is null) 
	{
		throw new Exception("Unable to find the recipe");
	}
	recipe.IsDeleted = true;
	await _context.SaveChangesAsync();
}
```
Также "мягкое удаление" - один из типичных сценариев применения [глобальных фильтров EF Core](https://docs.microsoft.com/ef/core/querying/filters)
