---
share: true
tags:
 - NET/ASPNETCore/testing
 - NET/EFCore/testing
 - testing
---
# Изоляция базы данных с помощью поставщика EF Core в памяти
В этом разделе рассмотрим, как писать тесты для кода, который полагается на экземпляр класса DbContext от EF Core. Мы разберем, как создать базу данных в памяти, какова разница между поставщиком EF в памяти и поставщиком SQLite в памяти, а также, как использовать поставщик SQLite для создания быстрых изолированных тестов.
Как уже [[ch-12-introducing-ef-core|говорилось]], EF Core — это инструмент объектно-реляционного отображения, который используется в основном с реляционными БД.
В следующем примере показана сильно урезанная версия класса `RecipeService` из [[ch-12-sample-application|приложения с рецептами]], показывающая единственный способ получить подробную информацию о рецепте:
```csharp
public class RecipeService
{
	readonly AppDbContext _context;
	public RecipeService(AppDbContext context)
	{
		_context = context;
	}
	public RecipeModel GetRecipe(int id)
	{
		return _context.Recipes
			.Where(x => x.RecipeId == id)
			.Select(x => new RecipeViewModel
				{
					Id = x.RecipeId,
					Name = x.Name
				})
			.SingleOrDefault();
	}
}
```
Для того, чтобы протестировать этот код, нужен *тестовый поставщик*, чтобы избавиться от необходмости использовать реальную БД в тестах.
У Microsoft есть два предназначенных для этих целей поставщика БД в памяти, а именно:
- *Microsoft.EntityFrameworkCore.InMemory* — этот поставщик хранит объекты непосредственно в памяти. При этом он не является реляционной БД, не соблюдает ограничения (*constraints*) и к ней нельзя выполнить настоящий SQL-запрос. Но при этом работает быстро;
- *Microsoft.EntityFrameworkCore.Sqlite* — реляционная БД SQLite. Обычно она пишет в файл, но у поставщика есть режим in-memory.

Вот как работают оба этих поставщика (в сравнении с Sql Server).
![[Pasted image 20221125135345.png]]
Далее будем использовать поставщик SQLite.
Прежде всего нужно добавить пакет Microsoft.EntityFrameworkCore.Sqlite в тестовый проект, который добавит метод расширения `UseSqlite()`.
Далее мы создаём объект `SqliteConnection` и используем строку подключения `"DataSource=:memory:"`, после чего открываем соединение.
> [!warning] Предупреждение
> База данных в памяти уничтожается, когда соединение закрыто. Если соединение не открыть самостоятельно, EF Core закроет его при удалении `DbContext`, поэтому, если мы хотим использовать БД несколькими экземплярами `DbContext`, нужно открывать соединение явно.

```csharp
[Fact]
public void GetRecipeDetails_CanLoadFromContext()
{
	var connection = new SqliteConnection("DataSource=:memory:");
	connection.Open();
	
	var options = new DbContextOptionsBuilder<AppDbContext>()
		.UseSqlite(connection)
		.Options;
	
	using(var context = new AppDbContext(options))
	{
		context.Database.EnsureCreated();
		context.Recipes.AddRange(
			new Recipe { RecipeId = 1, Name = "Recipe1" },
			new Recipe { RecipeId = 2, Name = "Recipe2" },
			new Recipe { RecipeId = 3, Name = "Recipe3" });
		context.SaveChanges();
	}
	using (var context = new AppDbContext(options))
	{
		var service = new RecipeService(context);
		var recipe = service.GetRecipe(id: 2);
		Assert.NotNull(recipe);
		Assert.Equal(2, recipe.Id);
		Assert.Equal("Recipe2", recipe.Name);
	}
}
```
Вот типичные шаги для тестирования класса, зависящего от DbContext:
1. создайте `SqliteConnection` со строкой подключения `"DataSource=:memory:"` и откройте соединение;
2. создайте `DbContextOptionsBuilder<>` и вызовите метод `UseSqlite()`, передавая открытое соединение;
3. извлеките объект `DbContextOptions` и свойства `Options`;
4. передайте параметры экземпляру `DbContext` и убедитесь, что БД соответствует модели EF Core при помощи `context.Database.EnsureCreated()`. Это действие аналогично запуску миграций в БД, но его следует выполнять *только* для тестовых БД. Создайте и добавьте необходимые тестовые данные и вызовите метод `SaveChanges()`, чтобы сохранить их;
5. Создайте *новый* экземпляр `DbContext` и внедрите его в тестируемый класс.

Использование двух отдельных экземпляров `DbContext` позволяет избежать ошибок в тестах из-за кэширования данных EF Core. При такмо подходе можно быть уверенным, что мы считали данные, сохраненные в БД.



