---
share: true
tags:
 - NET/ASPNETCore/filters
---
# Фильтры ресурсов: прерывание выполнения методов действий
Фильтры ресурсов - [[ch-13-mvc-filter-pipeline|первые]] фильтры общего назначения в конвейере фильтров MVC.

ASP.NET Core включает в себя несколько реализаций фильтров ресурсов, например:
- `ConsumesAttribute` — может использоваться для ограничения разрешенных форматов, которые может принимать метод действия, например `[Consumes("application/json")]`, при этом запросы с неподдерживаемым форматом получат ответ `415 Unsupported Media Type`;
- `DisableFormValueModelBindingAttribute` — предотвращает [[model-binding|привязку модели]] к данным формы в теле запроса. Это может быть полезно, если вы знаете, что метод действия будет обрабатывать загрузку крупных файлов, которой нужно будет управлять вручную[^1].

В [[ch-13-creating-custom-filters-for-your-application|коде контроллера]] есть строки, которые можно переделать в фильтр ресурсов, а именно
```csharp
if (!IsEnabled) { return BadRequest(); }
```
Это — *переключатель функций (feature toggle)*, который можно использовать, чтобы управлять доступностью API на основе настройки или значения из БД[^2]. В примере у нас — жёстко зашитое значение.

Вот примерная реализация `FeatureEnabledAttribute`:
```csharp
public class FeatureEnabledAttribute : Attribute, IResourceFilter
{
	public bool IsEnabled { get; set; }
	public void OnResourceExecuting(ResourceExecutingContext context)
	{
		if (!IsEnabled)
		{
			context.Result = new BadRequestResult();
		}
	}
	public void OnResourceExecuted(ResourceExecutedContext context) { }
}
```
В этом примере демонстрируются следующие концепции:
- фильтр может быть атрибутом. Им можно декорировать контроллер, методы действий и страницы Razor с помощью `[FeatureEnabled(IsEnabled=true)]`;
- интерфейс фильтра состоит из двух методов, и оба нужно реализовать, даже если нужен только один;
- методы фильтра получают объект контекста, обеспечивающего, помимо прочего, доступ к `HttpContext`;
- чтобы прервать выполнение конвейера, нужно задать `context.Result`. Фреймворк выполнит этот результат, игнорируя оставшиеся фильтры в конвейере и пропуская целиком метод действия.

[^1]: Подробнее о загрузке файлов см. [здесь](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads?view=aspnetcore-5.0)
[^2]: Подробнее см. в [блоге автора](https://andrewlock.net/series/adding-feature-flags-to-an-asp-net-core-app/)