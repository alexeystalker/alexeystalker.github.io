---
share: true
tags:
 - NET/EFCore/migrations
---
# Добавляем вторую миграцию
Большинство приложений неизбежно развивается, и изменение [[schema|схемы]] при этом весьма вероятно. Миграции EF Core упрощают эти процессы.
Предположим, что в нашем [[ch-12-sample-application|приложении]] мы решили добавить свойства `IsVegetarian` и `IsVegan` в сущность `Recipe`:
```csharp
public class Recipe
{
	public int RecipeId { get; set; }
	public string Name { get; set; }
	public TimeSpan TimeToCook { get; set; }
	public bool IsDeleted { get; set; }
	public string Method { get; set; }
	public bool IsVegetarian { get; set; }
	public bool IsVegan { get; set; }
	public ICollection<Ingredient> Ingredients { get; set; }
}
```
После изменения кода необходимо обновить внутреннее представление модели данных путем вызова команды
```bash
dotnet ef migrations add ExtraRecipeFields
```
Создастся вторая миграция путем добавления файла миграции, файла копии и обновления файла AppDbContextModelSnapshot.cs.
После чего можно применить миграцию, выполнив команду
```bash
dotnet ef database update
```
