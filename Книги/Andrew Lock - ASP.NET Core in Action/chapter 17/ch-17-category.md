---
share: true
tags:
 - NET/logging
---
# Категория сообщения: какой компонент создал журнал
В отличие от уровня, категория задаётся при создании экземпляра `ILogger`. Категория записывается в каждое сообщение журнала.
Категория — это объект типа `string`, так что ей можно задать любое значение, однако по соглашению в категорию помещают полное имя типа, используещего `ILogger`.
Одним из способов задать категорию будет внедрение `ILogger<T>` с указанием текущего типа, как это сделано [[ch-17-adding-log-messages-to-app|здесь]].
Альтернативный вариант — внедрять фабрику `ILoggerFactory` и задавать категорию явно:
```csharp
public class RecipeService
{
	private readonly ILogger _log;
	public RecipeService(ILoggerFactory factory)
	{
		_log = factory.CreateLogger("RecipeApp.RecipeService");
	}
}
```
Еще один вариант — использовать перегруженный фабричный метод:
```csharp
_log = factory.CreateLogger<RecipeService>();
```
В этом случае в переменной `_log` будет не просто `ILogger`, а `ILogger<RecipeService>`.
> [!tip] Совет
> Если вы не используете собственные категории, предпочтите внедрение `ILogger<T>` внедрению `ILoggerFactory`.